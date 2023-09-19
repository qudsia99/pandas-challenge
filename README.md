# pandas-challenge
===================


# Dependencies and Setup
import pandas as pd
from pathlib import Path

----------
#File to Load
school_data_to_load = Path("Resources/schools_complete.csv")
student_data_to_load = Path("Resources/students_complete.csv")


#Read School and Student Data File and store into Pandas DataFrames
school_data = pd.read_csv(school_data_to_load)
student_data = pd.read_csv(student_data_to_load)

----------
#Note: Debugging after running a function or variable helps keep the code clean along the way, and is optimal for minimizing bugs.

#Combine the data into a single dataset.  
merged = pd.merge(student_data, school_data, how="left", on=["school_name", "school_name"])
merged.head()

#Calculate the total number of unique schools
school_count = merged["school_name"].nunique() 
school_count

#Calculate the total number of students
student_count = merged["student_name"].count()
student_count

#Calculate the total budget
total_budget = merged["budget"].sum()
total_budget

#Calculate the average (mean) math score
average_math_score = merged["math_score"].mean() 
average_math_score

#Calculate the average (mean) reading score
average_reading_score = merged["reading_score"].mean() 
average_reading_score

# Use the following to calculate the percentage of students who passed math (math scores greather than or equal to 70)
passing_math_count = merged[(merged["math_score"] >= 70)].count()["student_name"] #Placing the ["student_name"] after count, narrows the results to only show us for that column.
passing_math_percentage = passing_math_count / float(student_count) * 100
#student count must be set as float in order to apply mathematical function (converts out of string)
passing_math_percentage

#Calculate the percentage of students who passed reading (hint: look at how the math percentage was calculated)  
passing_reading_count = merged[(merged["reading_score"]>=70)].count()["student_name"]
passing_reading_percentage = passing_reading_count/float(student_count)*100
passing_reading_percentage

#Use the following to calculate the percentage of students that passed math and reading
passing_math_reading_count = merged[(merged["math_score"] >= 70) & (merged["reading_score"] >= 70)
                                   ].count()["student_name"]
overall_passing_rate = passing_math_reading_count /  float(student_count) * 100
overall_passing_rate

# Create a high-level snapshot of the district's key metrics in a DataFrame
district_summary = pd.DataFrame([{"Total Schools":school_count,"Total Students":student_count, "Total Budget":total_budget,
                           "Average Math Score":average_math_score,"Average Reading Score":average_reading_score,
                           "% Passing Math":passing_math_percentage,"%Passing Reading":passing_reading_percentage,
                           "%Overall Passing":overall_passing_rate}])
#using a dictionary is efficient here as we are only interested in viewing a single row with total valued columns

#Formatting
district_summary["Total Students"] = district_summary["Total Students"].map("{:,}".format)
district_summary["Total Budget"] = district_summary["Total Budget"].map("${:,.2f}".format)

#Display the DataFrame
district_summary

# Use the code provided to select all of the school types
#Setting school_name as index makes the result more coherent and simple when viewing the results

school_types = school_data.set_index(["school_name"])["type"]
school_types


#Calculate the total student count per school
#This can be extracted using value_counts as we are measuring how many students (representing each row), are 
#in each school(representing each value in 'school_name' column)

per_school_counts = merged["school_name"].value_counts()
per_school_counts

#Calculate the total school budget and per capita spending per school
#Note: Don't forget to debug along the way :)

per_school_budget = school_data[["school_name","budget"]]
per_school_budget

per_school_budget = merged.groupby(["school_name"])["budget"].mean()
per_school_budget

per_school_capita = per_school_budget / per_school_counts
per_school_capita

#Calculate the average test scores per school
#To get a specific column, in an order determined by another column, groupby() is the most ideal function to use here
#Setting school_name as the groupby column allows us to view math means in each school

per_school_math = merged.groupby(["school_name"])["math_score"].mean()
per_school_math

per_school_reading = merged.groupby(["school_name"])["reading_score"].mean()
per_school_reading

#Calculate the number of students per school with math scores of 70 or higher
#Using a nested function with conditional, groupby and count

school_students_passing_math = merged[merged["math_score"] >= 70].groupby("school_name")["student_name"].count()
school_students_passing_math


students_passing_math = school_students_passing_math.sum()
students_passing_math

#Calculate the number of students per school with reading scores of 70 or higher
#Using the same nested formula structure in order to get reading scores >= 70

school_students_passing_reading = merged[merged["reading_score"]>=70].groupby("school_name")["student_name"].count()
school_students_passing_reading

students_passing_reading = students_passing_reading.sum()
students_passing_reading

# Use the provided code to calculate the number of students per school that passed both math and reading with scores of 70 or higher
#This requires us to use '&' to join two conditions, to get a retun value if met

students_passing_math_and_reading = merged[(merged["reading_score"] >= 70) & (merged["math_score"] >= 70)].groupby("school_name")["student_name"].count()
students_passing_math_and_reading

school_students_passing_math_and_reading = students_passing_math_and_reading.sum()
school_students_passing_math_and_reading

#Use the provided code to calculate the passing rates
#Finding percentage values >> (numerator/denominator)*100

per_school_passing_math = school_students_passing_math / per_school_counts * 100
per_school_passing_math
per_school_passing_reading = school_students_passing_reading / per_school_counts * 100
per_school_passing_reading
overall_passing_rate = students_passing_math_and_reading / per_school_counts * 100
overall_passing_rate

# Create a DataFrame called `per_school_summary` with columns for the calculations above.
per_school_summary = pd.DataFrame(merged.groupby("school_name")["student_name"].count())

#Adding all new columns and their values into new dataframe
per_school_summary["School Type"]= school_types
per_school_summary["Average Math Score"]= per_school_math
per_school_summary["Average Reading Score"]= per_school_reading


per_school_summary = per_school_summary.rename(columns={"student_name":"Total Students"})
#Renaming column for clarity

per_school_summary["Total School Budget"]= per_school_budget
per_school_summary["Per Student Budget"]= per_school_capita                    

per_school_summary["% Passing Math"] = per_school_passing_math
per_school_summary["% Passing Reading"] = per_school_passing_reading
per_school_summary["% Overall Passing"] = overall_passing_rate

#Formatting
per_school_summary["Total School Budget"] = per_school_summary["Total School Budget"].map("${:,.2f}".format)
per_school_summary["Per Student Budget"] = per_school_summary["Per Student Budget"].map("${:,.2f}".format)

#Display the DataFrame
per_school_summary.head()

#Sort the schools by `% Overall Passing` in descending order and display the top 5 rows.
top_schools = per_school_summary.sort_values("% Overall Passing", ascending=False)
top_schools.head(5)

#Sort the schools by `% Overall Passing` in ascending order and display the top 5 rows.
bottom_schools = per_school_summary.sort_values("% Overall Passing",ascending=True)
bottom_schools.head(5)

#Use the code provided to separate the data by grade
ninth_graders = merged[(merged["grade"] == "9th")]
tenth_graders = merged[(merged["grade"] == "10th")]
eleventh_graders = merged[(merged["grade"] == "11th")]
twelfth_graders = merged[(merged["grade"] == "12th")]


#Group by `school_name` and take the mean of the `math_score` column for each.
#Groupby is once again very efficient and effective in organizing the data

ninth_grade_math_scores = merged[merged["grade"]=="9th"].groupby("school_name")["math_score"].mean()
ninth_grade_math_scores

tenth_grader_math_scores = merged[merged["grade"]=="10th"].groupby("school_name")["math_score"].mean()
tenth_grader_math_scores
                                  
eleventh_grader_math_scores = merged[merged["grade"]=="11th"].groupby("school_name")["math_score"].mean()
eleventh_grader_math_scores

twelfth_grader_math_scores = merged[merged["grade"]=="12th"].groupby("school_name")["math_score"].mean()
twelfth_grader_math_scores


# Combine each of the scores above into single DataFrame called `math_scores_by_grade`
math_scores_by_grade = pd.DataFrame(merged.groupby("school_name")['student_name'].count())
math_scores_by_grade = math_scores_by_grade.rename(columns={"student_name":"Total Students"})                               

math_scores_by_grade["9th Grade Math Average"]= ninth_grade_math_scores 
math_scores_by_grade["10th Grade Math Average"]= tenth_grader_math_scores 
math_scores_by_grade["11th Grade Math Average"]= eleventh_grader_math_scores 
math_scores_by_grade["12th Grade Math Average"]= twelfth_grader_math_scores 



#Minor data wrangling
#Dropping irrelevant column: 'total students'
math_scores_by_grade = math_scores_by_grade[["9th Grade Math Average", 
                                                   "10th Grade Math Average", 
                                                   "11th Grade Math Average", 
                                                   "12th Grade Math Average"]]
math_scores_by_grade.index.name = None


#Display the DataFrame
math_scores_by_grade


#The mean math scores across all schools, by grade. Just in case to refer to when debugging.
#78.93565918653576 Grade.9 Math Scores
#78.94148308418568 Grade.10 Math Scores
#79.08354822073234 Grade.11 Math Scores
#78.99316369160653 Grade.12 Math Scores

#Use the code provided to separate the data by grade

ninth_graders = merged[(merged["grade"] == "9th")]
tenth_graders = merged[(merged["grade"] == "10th")]
eleventh_graders = merged[(merged["grade"] == "11th")]
twelfth_graders = merged[(merged["grade"] == "12th")]

#Group by `school_name` and take the mean of the the `reading_score` column for each.

ninth_grade_reading_scores = merged[merged["grade"]=="9th"].groupby("school_name")["reading_score"].mean()
ninth_grade_reading_scores

tenth_grader_reading_scores = merged[merged["grade"]=="10th"].groupby("school_name")["reading_score"].mean()
tenth_grader_reading_scores

eleventh_grader_reading_scores = merged[merged["grade"]=="11th"].groupby("school_name")["reading_score"].mean()
eleventh_grader_reading_scores

twelfth_grader_reading_scores = merged[merged["grade"]=="12th"].groupby("school_name")["reading_score"].mean()
twelfth_grader_reading_scores

# Combine each of the scores above into single DataFrame called `reading_scores_by_grade`

reading_scores_by_grade = pd.DataFrame(merged.groupby("school_name")['student_name'].count())
reading_scores_by_grade = reading_scores_by_grade.rename(columns={"student_name":"Total Students"})

reading_scores_by_grade["9th Grade Reading Average"] = ninth_grade_reading_scores
reading_scores_by_grade["10th Grade Reading Average"] = tenth_grader_reading_scores
reading_scores_by_grade["11th Grade Reading Average"] = eleventh_grader_reading_scores
reading_scores_by_grade["12th Grade Reading Average"] = twelfth_grader_reading_scores


#Minor data wrangling
#Dropping 'total student' column, due to irrelevance
reading_scores_by_grade = reading_scores_by_grade[["9th Grade Reading Average", 
                                                   "10th Grade Reading Average", 
                                                   "11th Grade Reading Average", 
                                                   "12th Grade Reading Average"]]
reading_scores_by_grade.index.name = None

#Display the DataFrame
reading_scores_by_grade

#Establish the bins 
#Setting bin values can be decided when viewing 'merged' or per_school_capita to see what the range of per_capita spending is.
spending_bins = [0, 585, 630, 645, 680]
labels = ["<$585", "$585-630", "$630-645", "$645-680"]

#Create a copy of the school summary since it has the "Per Student Budget" 
school_spending_df = per_school_summary.copy()

# Use `pd.cut` to categorize spending based on the bins.
pd.cut(per_school_capita,spending_bins, labels=labels)

school_spending_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita ,spending_bins, labels=labels)
school_spending_df

#Calculate averages for the desired columns. 
spending_math_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Math Score"].mean()
spending_reading_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Reading Score"].mean()
spending_passing_math = school_spending_df.groupby(["Spending Ranges (Per Student)"])["% Passing Math"].mean()
spending_passing_reading = school_spending_df.groupby(["Spending Ranges (Per Student)"])["% Passing Reading"].mean()
overall_passing_spending = school_spending_df.groupby(["Spending Ranges (Per Student)"])["% Overall Passing"].mean()

#Assemble into DataFrame
spending_summary = pd.DataFrame(school_spending_df.groupby("Spending Ranges (Per Student)")["Total Students"].count())

spending_summary["Average Math Score"] = spending_math_scores
spending_summary["Average Reading Score"] = spending_reading_scores 
spending_summary["% Passing Math"] = spending_passing_math
spending_summary["% Passing Reading"] = spending_passing_reading
spending_summary["% Overall Passing"] = overall_passing_spending

spending_summary = spending_summary[["Average Math Score","Average Reading Score",
                                    "% Passing Math","% Passing Reading","% Overall Passing"]]

#Display results
spending_summary

#Establish the bins.
size_bins = [0, 1000, 2000, 5000]
labels = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]

#Categorize the spending based on the bins
#Use `pd.cut` on the "Total Students" column of the `per_school_summary` DataFrame.
pd.cut(per_school_summary["Total Students"], size_bins, labels=labels)

per_school_summary["School Size"] = pd.cut(per_school_summary["Total Students"], size_bins, labels=labels)
per_school_summary

#Calculate averages for the desired columns. 
size_math_scores = per_school_summary.groupby(["School Size"])["Average Math Score"].mean()
size_reading_scores = per_school_summary.groupby(["School Size"])["Average Reading Score"].mean()
size_passing_math = per_school_summary.groupby(["School Size"])["% Passing Math"].mean()
size_passing_reading = per_school_summary.groupby(["School Size"])["% Passing Reading"].mean()
size_overall_passing = per_school_summary.groupby(["School Size"])["% Overall Passing"].mean()

# Create a DataFrame called `size_summary` that breaks down school performance based on school size (small, medium, or large).
#Use the scores above to create a new DataFrame called `size_summary`
size_summary = pd.DataFrame(per_school_summary.groupby("School Size")["Total Students"].count())

size_summary["Average Math Score"] = size_math_scores
size_summary["Average Reading Score"] = size_reading_scores                 
size_summary["% Passing Math"] = size_passing_math
size_summary["% Passing Reading"] = size_passing_reading
size_summary["% Overall Passing"] = size_overall_passing  

size_summary = size_summary[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","% Overall Passing"]]
size_summary.index.name = None
#Display results
size_summary

# Group the per_school_summary DataFrame by "School Type" and average the results.
average_math_score_by_type = per_school_summary.groupby(["School Type"])["Average Math Score"].mean()
average_reading_score_by_type = per_school_summary.groupby(["School Type"])["Average Reading Score"].mean()
average_percent_passing_math_by_type = per_school_summary.groupby(["School Type"])["% Passing Math"].mean()
average_percent_passing_reading_by_type = per_school_summary.groupby(["School Type"])["% Passing Reading"].mean()
average_percent_overall_passing_by_type = per_school_summary.groupby(["School Type"])["% Overall Passing"].mean()

#Assemble the new data by type into a DataFrame called `type_summary`
type_summary = pd.DataFrame(per_school_summary.groupby("School Type")["Total Students"].count())

type_summary["Average Math Score"] = average_math_score_by_type
type_summary["Average Reading Score"] = average_reading_score_by_type
type_summary["% Passing Math"] = average_percent_passing_math_by_type
type_summary["% Passing Reading"] = average_percent_passing_reading_by_type
type_summary["% Overall Passing"] = average_percent_overall_passing_by_type

type_summary = type_summary[["Average Math Score","Average Reading Score",
                             "% Passing Math","% Passing Reading","% Overall Passing"]]
#Display results
type_summary
