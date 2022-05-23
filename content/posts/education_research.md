---
title: "Does College of Study Effect Graduate Student Success?"
date: 2022-04-25
summary: An overview of my undergraduate research experience. 
showtoc: true
draft: false
---
*Special thanks to my research team as well as the mentors that provided support.* 

## Main Idea 
Our project sought to identify whether graduate student success at a large research university differed by the college of study. Here, we define success as graduation with the intended degree.

## Purpose
The literature that we read discussed research on trends for graduate student success, in both Master’s and doctoral programs, for student success and demographic factors. For student success, most studies cited student characteristics as being the most influential factor. One study analyzed rates and reasons for attrition across different disciplines using student and faculty interviews. That study found that across disciplines, 34% of students cited personal reasons as the primary factor for attrition while a different study found that personal factors such as gender, family status, age, and part-time status were most significant in the decision to stay or go (Source 1, Source 4). Additionally, the gender gap in STEM doctoral programs has been decreasing since the 1970s, with the proportion of women peaking at 28% in 2009, though there does not seem to be a difference in graduation rates between genders (Source 2). In contrast, while the number of underrepresented minority (URM) students receiving bachelor’s degrees has been increasing, the number of URM students receiving graduate degrees has stayed relatively consistent (Source 6).

Attrition rates are a big problem for graduate programs in the US with approximately 57% of students leaving their graduate program each year without a degree (Source 4). In a study at a large research institution in the US, the programs with the highest attrition rates were English, mathematics, and electrical and computer engineering while the programs with the lowest attrition rates were communication, oceanography, and psychology (Source 4). Overall, most of these sources utilized qualitative interviews in order to generate common responses for attrition reasons, value of minority workshops, and student success factors (Sources 4, 5, 6). Although they compared means within each categorical group, we did not find any trends regarding common analysis types, such as linear regression or hypothesis testing, that we can apply to our project. 

After completing this literature review, there are still some gaps that we have found in the literature. For instance, most of the sources focus on doctoral graduate programs, not Master’s, and we could not find any sources that compared Master's to Ph.D. student completion rates. Additionally, most of these sources were qualitative in nature, so they did not detail their statistical analysis methods, such as regression modeling, that would be applicable to this project. Finally, there were no studies that compared student success rates by college. One study found the programs that had the lowest median time to degree and another found the programs that had the three highest and three lowest completion rates, but no study compared these variables across all programs (Source 5, Source 4). 

## Methodology
Our data comes from a large research institution in the US and was collected between 2009 and 2021. For this project, we decided to measure “success” as completing the intended degree program and measure “failure” as leaving the university with no degree. We removed observations of students who were still “active in program” since we do not know if these students would complete their degree program or drop out. We also removed students who are in more than one college (approximately 1% of the data set), students who begin in one degree path and leave with a different degree (ex, Ph.D. to Master’s) as well as those who had a GPA = 0 but had completed their degree. Additionally, we counted students who switched colleges as only being part of their most recent college. For example, if a student switched from college 9 to college 10 and graduated, then they would be counted as a success for 10 but excluded from 9.

We fit two logistic regression models - one for Master’s students and one for Ph.D. students - to study the likelihood of graduation based on college, controlling for confounding variables such as GPA, gender, age, and underrepresented minority status. In these models, our response variable was success (graduation) or failure (failing to complete their intended program). We used the following predictor variables: College the student is a part of (categorical variable), Cumulative GPA (quantitative variable), URM status (categorical variable), Gender (categorical variable), and Age (quantitative variable). We decided to use these additional predictor variables in addition to the college in which the student is studying in order to control for confounding variables that may have similar effects across colleges. 

## Findings
The rate of student degree completion by department must be analyzed after completing the exploratory data analysis. We found that even at the college level, graduation rates differ by degree program. By degree program, we mean whether the student is pursuing a Master’s Degree (MS, MA, MR, etc) or a Doctorate (Ph.D., EDD). Across all colleges, Master’s students have an average graduation rate of approximately 86%, whereas doctoral students have an average graduation rate of approximately 65%. These values are averages and do not capture the variability of graduation rates between different colleges.

After creating our logistic regression models, our main takeaway is that college of study is a significant factor in the probability of graduation, even when controlling for other potentially confounding variables [Wald’s Chi-Square = 308 (Master’s) and 38 (Ph.D.); p-value for both < 0.0001]. In order to expand upon our results, a next step would be to look into colleges with lower success rates and explore ways to improve the success rates of those colleges. Future research is necessary to validate these findings and improve specificity, such as studying the student’s department instead of just their college. 

## Presentation
![Research Poster](/final_poster.png)

## References

| Source Number | Journal Title | Link |
|---------------|---------------|------|
| 1 | Fitting the Mold of Graduate School: A Qualitative Study of Socialization in Doctoral Education | https://link.springer.com/article/10.1007/s10755-008-9068-x |
| 2 | The bachelor’s to Ph.D. STEM pipeline no longer leaks more women than men: a 30-year analysis | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4331608/ | 
| 3 | Predicting Academic Success in Master's Degree Programs in Education | https://www.jstor.org/stable/27531822?pq-origsite=summon&seq=1#metadata_info_tab_contents | 
| 4 | Student and faculty attributions of attrition in high and low-completing doctoral programs in the United States | https://link.springer.com/article/10.1007/s10734-008-9184-7 | 
| 5 | Departmental Factors Affecting Time-to-Degree and Completion Rates of Doctoral Students at One Land-Grant Research Institution | https://www.jstor.org/stable/2649335?seq=1#metadata_info_tab_contents | 
| 6 | Strategies  for  Multicultural  StudentSuccess:  What  About  Grad  School? | https://onlinelibrary.wiley.com/doi/epdf/10.1002/j.2161-0045.2006.tb00200.x | 

