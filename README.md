# NYS COVID19 Disparities Survey

This is a repository of the NYS COVID19 Disparities Survey. The following is the information of the survey.
* Time Period: Nov. 23. 2020 - Dec. 8. 2020.
* Sample Information
  * Size: 1,353 (based on Dec-11th version).
  * Ages 18 or higher
  * Area: New York State
* Method: Qualtrics Online survey.

## File Descriptions

* Raw Data
  - **NYS_COVID19_Disparities_2020_12_11_Numeric.csv**: Numeric version, downloaded on Dec. 11. 2020.
  - **NYS_COVID19_Disparities_2020_12_11_Text.csv**: Text version, downloaded on Dec. 11. 2020.
  - **NYS+Coronavirus+Disparities+Study_December+7,+2020_14.25**: Survey data with text values (Original file from Elizabeth from Qualtrics).
 
* STATA Files
  - **NYS_COVID19_Disparities.dta**: A cleaned dta file. This was created by using "NYS_COVID19_Disparities_2020_12_11_Numeric.csv."
    - This file includes some newly created variables. These are:
      - EXP1: 0 if Broader risk group arm; 1 if Racial disparities arm
      - RACE: 0 "White" 1 "Black" 2 "Others"
      - HISPANIC: 0 "Non Hispanic" 1 "Hispanic" (recoded from DEMO_14_1: 0 "Non Hispanic" 1 "Puerto Rican" 1 "Mexican" 2 "Dominican" 3 "Cuban" 4 "Other Hispanic")
      - METRO: 1 if the zipcode is included in the metropolitan area (New York (100-102), Bronx (104), Kings (112), and Queens (11004, 11005, 111, 114-116))
      - AGE: 0 "18-29" 1 "30-39" 2 "40-49" 3 "50-59" 4 ">60"
      - VACC_TAKE: 0 "Get the vaccine" 1 "Don't get the vaccine"
      - VACC_TAKE3: 1 "Definitely get the vaccine" 2 "Prabably get or don't got" 3 "Definitely don't get the vaccine"
      - BELIEVE_MASK: 0 "No/Not sure" 1 "Yes"
      - WORRY_CONTRACTING: 0 "Not worried at all" 1 "Worried"
      - MEDIA_FREQ: 0 "< Weekely" 1 "Weekly" 2 "Daily" 3 "> Daily"
      - MEDIA_FOX: 0 "Others" 1 "Fox News Viewer"
    - Update History
      - 2020/12/12: New variables (VACC_TAKE, VACC_TAKE3, BELIEVE_MASK, WORRY_CONTRACTING, MEDIA_FREQ, MEDIA_FOX); Variables revised (DEMO_14_1)
      - 2020/12/11: Initially created
  - **NYS_COVID19_Disparities.do**: STATA Do-file of "NYS_COVID19_Disparities.md." This includes the data cleaning procedures.
    - Update History
      - 2020/12/12: Data cleaning process revised
      - 2020/12/11: Initially created

* Review and Analysis
  - **NYS_COVID19_Disparities.md**: 
    - 2020/12/11: Added quality check and initial review based on racial categories


## Contributors
Principal Investigator: [Ashley Fox](https://twitter.com/ashfoxly)

Author: [@Yongjin Choi](https://twitter.com/TheYongjinChoi)
