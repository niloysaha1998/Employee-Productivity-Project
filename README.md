# Employee-Productivity-Project
Statistical methods to identify the key drivers of employee performance in an organisational environment.

## Project Title

### ***'Exploration of Factors that Effect Employee Productivity'***

```{r}
library(tidyverse)
library(dplyr)
library(car)
library(ggplot2)
library(corrplot)
library(readr)
library(plotly)
library(rgl)
library(scales)
library(gridExtra)
library(vcd)
```

## Background

In a fast-paced corporate world where employee productivity determines the success and sustainability of organizations, This research project titled "Exploration of Factors that Contribute to Employee Productivity" aims to identify key drivers in that. Inspired by the ongoing discourse around work-life balance and its impact on performance, we aimed to untangle the complex web of variables influencing productivity in the workplace. Leveraging a comprehensive dataset from Kaggle, we investigated key factors, including monthly salary, work hours, remote work frequency, team size, and training opportunities. This project has not only provided actionable insights into optimizing resource allocation but also challenged traditional assumptions about performance drivers.

## Dataset Description

```{r}


library(readr)
productivity <- read.csv("Extended_Employee_Performance_and_Productivity_Data.csv")
summary(productivity)
str(productivity)



```

**Source:**

The dataset can be accessed through Kaggle (Public library of useful datasets), titled *"Employee Performance and Producitvity Data." (link :* <https://www.kaggle.com/datasets/mexwell/employee-performance-and-productivity-data?>)

-   Rights: As the dataset is placed in public domain, there is no restrictions in sharing and usage.

-   Limitations: The data seems a little bit unrealistic. In a lot of visualizations, it shows a very even split between Gender, Job Titles or Performance Scores

**Size:**

Number of Rows: 100000

Number of Columns: 20

**key Features:**

The dataset contains the following key attributes:

-   Age: The current age of the employee, recorded in years

-   Monthly_Salary: The total monthly salary of the employee, recorded in the organization’s currency.

-   Years_At_Company: The number of years the employee has worked in the company.

-   Remote_Work_Frequency : The frequency of remote work allowed or performed by the employee, often measured as a percentage or number of days.

-   Work_Hours_Per_Week : The total hours worked per week by the employee, including overtime if applicable.

-   Team_Size The team size the employee is part of, indicating their immediate working group.

Other Demographic Details: Additional information such as Sick_days, Promotion, and Overtime_hours.

## Analysis

### Part 1: Data Preparation

```{r}

#chaning the column types
productivity$Hire_Date <- as.Date(productivity$Hire_Date, format = "%Y-%m-%d") #removing the Hour Minute Second because it is not relevant for our analysis
productivity$Age <- as.numeric(productivity$Age)
productivity$Employee_ID <- as.numeric(productivity$Employee_ID)
productivity$Performance_Score <- as.numeric(productivity$Performance_Score)
productivity$Work_Hours_Per_Week <- as.numeric(productivity$Work_Hours_Per_Week)
productivity$Projects_Handled <- as.numeric(productivity$Projects_Handled)
productivity$Overtime_Hours <- as.numeric(productivity$Overtime_Hours)
productivity$Sick_Days <- as.numeric(productivity$Sick_Days)
productivity$Remote_Work_Frequency <- as.numeric(productivity$Remote_Work_Frequency)
productivity$Team_Size <- as.numeric(productivity$Team_Size)
productivity$Promotions <- as.numeric(productivity$Promotions)

str(productivity)
```

```{r}
#NAs
sum(is.na(productivity))
# zero NAs
```

```{r}

# Check for duplicate rows
duplicates <- productivity[duplicated(productivity), ] # View duplicate rows, if any
productivity <- productivity %>% distinct() 

productivity

```

```{r}
# Removing Outliers and Cleaning the Dataset

removing_outliers <- function(productivity, cols) {
  for (col in cols) {
    
   
    Q1 <- quantile(productivity[[col]], 0.25, na.rm = TRUE)
    Q3 <- quantile(productivity[[col]], 0.75, na.rm = TRUE)
    IQR <- Q3 - Q1
    
  
    lower_bound <- Q1 - 1.5 * IQR
    upper_bound <- Q3 + 1.5 * IQR
    
    
    productivity <- productivity %>%
      filter(.data[[col]] >= lower_bound & .data[[col]] <= upper_bound)
  }
  return(productivity)
}

cols <- c("Age", "Work_Hours_Per_Week", "Overtime_Hours", "Remote_Work_Frequency", "Sick_Days","Years_At_Company", "Monthly_Salary", "Projects_Handled", "Training_Hours", "Employee_Satisfaction_Score")


final_cleaned_dataset <- removing_outliers(productivity, cols)

str(final_cleaned_dataset)


```

Comment: The CSV file was downloaded and subsequently imported into R Studio for preprocessing. During this stage, the column types were adjusted to appropriate formats, such as numeric and date. The date column was refined from the original "Year-Month-Day Hour-Minute-Second" format to "Year-Month-Day," as the time component (Hour-Minute-Second) was deemed irrelevant for the scope of this analysis. A thorough check for missing values (NAs) was conducted, confirming the absence of such entries.

### Part 2: Visualisation

```{r}
# getting an overview of the gender distribution
gender_counts <- table(final_cleaned_dataset$Gender)
print(gender_counts)
```

```{r}
# plotting 
gender_count <- final_cleaned_dataset %>%
  group_by(Job_Title, Gender) %>%
  summarise(count = n(), .groups = 'drop')

ggplot(gender_count, aes(x = Job_Title, y = count, fill = Gender)) + 
    geom_bar(stat = "identity", position = "dodge") + 
    labs(
        x = "Job Title",
        y = "Count of Gender"
    ) + 
    theme_minimal() +
    theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

```{r}
# plotting 
gender_count <- final_cleaned_dataset %>%
  group_by(Department, Gender) %>%
  summarise(count = n(), .groups = 'drop')

ggplot(gender_count, aes(x = Department, y = count, fill = Gender)) + 
    geom_bar(stat = "identity", position = "dodge") + 
    labs(
        x = "Department",
        y = "Count of Gender"
    ) + 
    theme_minimal() +
    theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

**Comment:** The distribution of gender appears balanced across job titles and departments, with roughly equal representation of males and females. This uniformity might indicate a conscious effort toward equal gender representation, although it also raises questions about dataset randomness or potential data constraints.

```{r}
#Plotting age and monthly salary

ggplot(final_cleaned_dataset, aes(x = Age, y = Monthly_Salary)) + 
    geom_point(color = "blue", alpha = 0.6) + 
    labs(
        title = "Age vs Monthly Salary",
        x = "Age",
        y = "Monthly Salary"
    ) + 
    theme_minimal()

```

**Comment:** The scatterplot shows no apparent trend or correlation between age and monthly salary. Salaries are uniformly distributed across all age groups, suggesting that factors other than age (e.g., job role, education, or performance) are likely driving compensation.

```{r}
#plotting performnce score over education level
ggplot(final_cleaned_dataset, aes(x = Education_Level, y = Performance_Score)) + 
    geom_point(color = "lightblue", alpha = 0.6) +
    labs(
        title = "Performance Score by Education Level",
        x = "Education Level",
        y = "Performance Score"
    ) + 
    theme_minimal() +
    theme(axis.text.x = element_text(angle = 45, hjust = 1))


```

**Comment:** Higher education levels (e.g., Master’s, Ph.D.) do not seem to guarantee higher performance scores, indicating that other factors (for example: training, work experience) may carry more weight in determining employee performance.

```{r}
#checking the count for every score
performance_counts <- table(final_cleaned_dataset$Performance_Score)
print(performance_counts)
```

### Part 3 : Correlation Analysis

```{r}
corr_1 <- final_cleaned_dataset[, c("Performance_Score", "Employee_Satisfaction_Score")]
cor_matrix1 <- cor(corr_1, use = "complete.obs", method = "pearson")
print(cor_matrix1)
```

```{r}
cor_2 <- final_cleaned_dataset[, c("Performance_Score", "Promotions")]
cor_matrix2 <- cor(cor_2)
print(cor_matrix2)
```

```{r}
cor_3 <- final_cleaned_dataset[, c("Performance_Score", "Monthly_Salary")]
cor_matrix3 <- cor(cor_3)
print(cor_matrix3)
```

```{r}
cor_4 <- final_cleaned_dataset[, c("Performance_Score", "Age")]
cor_matrix4 <- cor(cor_4)
print(cor_matrix4)
```

```{r}
cor_5 <- final_cleaned_dataset[, c("Performance_Score", "Work_Hours_Per_Week")]
cor_matrix5 <- cor(cor_5)
print(cor_matrix5)
```

```{r}
cor_6 <- final_cleaned_dataset[, c("Performance_Score", "Overtime_Hours")]
cor_matrix6 <- cor(cor_6)
print(cor_matrix6)
```

```{r}
library(ggcorrplot)
selected_columns <- final_cleaned_dataset[, c("Performance_Score", "Age", "Monthly_Salary", "Overtime_Hours", "Work_Hours_Per_Week", "Sick_Days", "Projects_Handled", "Remote_Work_Frequency", "Team_Size", "Employee_Satisfaction_Score")]

cor_matrix <- cor(selected_columns, use = "complete.obs")

ggcorrplot(cor_matrix, 
           method = "circle", 
           type = "lower", 
           lab = TRUE, 
           title = "Correlation Plot")

```

**Comment:** Correlation matrices were used to analyze relationships between variables, represented by Pearson correlation coefficients (ranging from -1 to 1). Most variables were independent, except for a notable positive correlation between Performance Score and Monthly Salary, indicating that higher performance scores are generally associated with increased salaries. This highlights the significant link between performance and compensation.

### Part 4 : Linear Regression

```{r}



#Regression Model Building
Model_employee <- lm(Performance_Score ~ Age + Gender + Resigned + Years_At_Company + Team_Size + Monthly_Salary + Work_Hours_Per_Week + Training_Hours + Projects_Handled+ Sick_Days+ + Promotions+ Overtime_Hours + Employee_Satisfaction_Score+  Remote_Work_Frequency + Education_Level ,data = final_cleaned_dataset)

summary(Model_employee)



```

**Comment:** A multiple regression analysis was conducted using Performance Score as the response variable and relevant predictors to identify factors influencing productivity. The results showed that only Monthly Salary, Team Size, and Master’s Degree (education level) were statistically significant, with p-values below 0.05. Higher salaries were strongly linked to better performance, making Monthly Salary the most influential factor. Employees with a Master’s degree also tended to perform better, while larger team sizes negatively impacted individual performance. However, the low R-squared value suggests these factors explain only a small portion of the variability in performance, indicating the need for further exploration of additional variables.

```{r}
#Refining the model for better fit based on results

refined_model_employee <- lm(Performance_Score ~ Monthly_Salary +Education_Level+  Team_Size, data = final_cleaned_dataset)
summary(refined_model_employee)

#Need to extract number values from the model

coeff_table_2 <- refined_model_employee$coefficients
rounded_coefficients_2 <- round(coeff_table_2, 2)
rounded_coefficients_2
plot(refined_model_employee)
vif(refined_model_employee)
```

**Comment:** A refined regression model was created using only statistically significant factors, yielding consistent results in terms of p-values and significance levels. However, the R-squared value remained unchanged, indicating no improvement in explanatory power. To validate the model, key assumptions were tested. The linearity test showed a slight curvature in the red smoothing line, suggesting potential non-linearity, while the non-uniform spread of residuals indicated possible heteroscedasticity. This was further confirmed by the scale-location plot, where residuals were unevenly distributed. The normality test revealed that residuals mostly followed a diagonal line but showed slight deviations at the tails, hinting at some degree of non-normality. Finally, the leverage point analysis identified a few high-leverage points on the extreme right, which could potentially skew the regression results.

```{r}
#Exploring interaction model

interaction_model_employee <- lm(Performance_Score ~ Monthly_Salary * Education_Level + Team_Size , data = final_cleaned_dataset)

summary(interaction_model_employee)
```

**Comment:** An interaction model was developed to examine whether the combination of high salary and high education level has a significant effect on performance. However, the analysis revealed no improvement in the model's R-squared value, indicating that the interaction terms did not enhance the model's explanatory power.

```{r}
#Exploring taking log of response variable
final_cleaned_dataset$log_performance <- log(final_cleaned_dataset$Performance_Score)
#refined_log_model
log_model <- lm(log_performance ~ Monthly_Salary +Team_Size +Education_Level , data = final_cleaned_dataset)
summary(log_model)
```

```{r}
# Exploring taking log of both sides
final_cleaned_dataset$log_monthly_salary <- log(final_cleaned_dataset$Monthly_Salary)

#refined_log_model_2
log_model_2 <- lm(log_performance ~ log_monthly_salary + Education_Level + Team_Size  , data = final_cleaned_dataset)
summary(log_model_2)
```

**Comment:** The model was further refined by applying a logarithmic transformation to both Performance Score and Monthly Salary, the most statistically significant factor. Despite this adjustment, the R-squared value remained low, indicating no improvement in the model's explanatory power.

## Results / Insights

This analysis concludes that the most reliable regression model for this dataset identifies Monthly Salary, Team Size, and Education Level as the strongest predictors of employee performance, with Monthly Salary being the most significant. Despite attempts to refine the model through transformations and interaction terms, no improvements in explanatory power were observed, reinforcing this model as the best representation of the data. These findings highlight the critical influence of compensation, team dynamics, and education on employee productivity. The analysis also underscores the importance of addressing underlying data complexities and exploring additional factors to enhance the predictive strength of the models.


