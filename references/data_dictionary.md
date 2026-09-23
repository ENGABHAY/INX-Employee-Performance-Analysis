# Data Dictionary — INX Future Inc. Employee Performance Dataset

Source: `INX_Future_Inc_Employee_Performance_CDS_Project2_Data_V1_8.xls` (1200 rows × 28 columns)

The **"Knowable at hiring?"** column reflects the split used in `src/Data
Processing/data_processing.ipynb` and `src/models/train_model.ipynb`: only columns marked
**Yes** are used by the Hiring Model, since a candidate has no employment history at INX yet.

| Column | Description | Knowable at hiring? |
|---|---|---|
| EmpNumber | Unique employee identifier (dropped before modeling) | — |
| Age | Employee age in years | Yes |
| Gender | Male / Female | Yes |
| EducationBackground | Field of educational background | Yes |
| MaritalStatus | Single / Married / Divorced | Yes |
| EmpDepartment | Department the employee belongs to | Yes |
| EmpJobRole | Specific job role/title | Yes |
| BusinessTravelFrequency | How often the employee travels for business | Yes |
| DistanceFromHome | Distance from home to office | Yes |
| EmpEducationLevel | Education level, ordinal 1 (lowest) – 5 (highest) | Yes |
| EmpEnvironmentSatisfaction | Workplace environment satisfaction, ordinal 1–4 | No |
| EmpHourlyRate | Hourly pay rate | No |
| EmpJobInvolvement | Job involvement rating, ordinal 1–4 | No |
| EmpJobLevel | Seniority/job level, ordinal 1–5 | Yes |
| EmpJobSatisfaction | Job satisfaction rating, ordinal 1–4 | No |
| NumCompaniesWorked | Number of companies worked at previously | Yes |
| OverTime | Whether the employee regularly works overtime (Yes/No) | No |
| EmpLastSalaryHikePercent | Percentage of the most recent salary hike | No |
| EmpRelationshipSatisfaction | Relationship satisfaction at work, ordinal 1–4 | No |
| TotalWorkExperienceInYears | Total years of professional work experience | Yes |
| TrainingTimesLastYear | Number of trainings attended last year | No |
| EmpWorkLifeBalance | Work-life balance rating, ordinal 1–4 | No |
| ExperienceYearsAtThisCompany | Years of tenure at INX Future Inc. | No |
| ExperienceYearsInCurrentRole | Years in the current role | No |
| YearsSinceLastPromotion | Years since the employee's last promotion | No |
| YearsWithCurrManager | Years working under the current manager | No |
| Attrition | Whether the employee has left the company (Yes/No) | No |
| PerformanceRating | **Target variable.** 2 = Good, 3 = Excellent, 4 = Outstanding | — |

**12 columns** are hiring-safe (used by the Hiring Model); **14** are current-employment-only
(used, together with the 12, by the Diagnostic Model). No missing values or duplicate records
were found in this dataset (verified in `src/Data Processing/data_processing.ipynb`).
