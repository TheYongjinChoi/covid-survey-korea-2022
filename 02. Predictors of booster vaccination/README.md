# Who Gets COVID-19 booster vaccination? Trust in Public Health Institutions and Promotion Strategies Post-Pandemic in the Republic of Korea

*Paper Authors: Yongjin Choi Soohyun Park, Jinwoo Lee, Youngsung Kim, Byoung Joon Kim, Leesa Lin, and Ashley M. Fox* </br> *Script Author: Yongjin Choi* </br> *Last updated: Nov. 12. 2023*

* What's Included
  * [Part I. Basic Setting](#part-i-basic-setting)
  * [Part II. Data Prep](#part-ii-data-prep)
    - Cleaning
    - Weighting
  * [Part III. Data Review](#part-iii-data-review)
    - Weighted
    - Unweighted
  * [Part IV. Estimation](#part-iv-estimation)
    - Figure 1 & Appendix 1
    - Appendix 2
    - Appendix 3
    - Figure 2 & Appendix 4
    - Appendix 5
    - Appendix 6

## Part I. Basic Setting


```stata
********************************************************************************
/*----- Basic Setting -----*/
********************************************************************************

/*----- Essentials -----*/

clear all
qui global localpath "C:\Dropbox\05_Git\covid-survey-korea-2022\02. Predictors of booster vaccination"
cd "$localpath"
cap mkdir "$localpath\img"
qui global rawdata "C:\Dropbox\02_Data"
qui global outputs "$localpath\img"

* Output width
set linesize 240
display "{hline}"

* Color scheme for plots
colorpalette #176d90 #841618 #f18821 #3d7337 #a039b7 #009999 #cc6699 #ff6633 #cccc33 #9e6eac, globals
qui grstyle clear
qui set scheme s2color
qui grstyle init
qui grstyle set plain, horizontal grid
qui grstyle color background white
qui grstyle yesno draw_major_hgrid yes
qui grstyle yesno draw_major_ygrid yes
qui grstyle color major_grid none
qui grstyle linepattern major_grid solid
qui grstyle linewidth major_grid vvthin
qui grstyle set legend 4, box inside
qui grstyle color ci_area gs12%50
```

### Packages dependency


```stata
/*----- Installing Packages -----*/
ssc install estout, replace
ssc install coefplot, replace
ssc install grstyle, replace
ssc install palettes, replace
ssc install colrspace, replace
```

# Part II. Data Prep


```stata
********************************************************************************
/*----- Part II. Data Prep -----*/
********************************************************************************
```

## Cleaning


```stata
import delimited "$rawdata\01_Survey\2022_Korea Vaccine Survey\HBK_Public\HBK_W1_20220510.csv", clear
gen WAVE = 1
save "HBK_W1", replace
import delimited "$rawdata\01_Survey\2022_Korea Vaccine Survey\HBK_Public\HBK_W2_20220707.csv", clear
gen WAVE = 2
save "HBK_W2", replace

import delimited "$rawdata\01_Survey\2022_Korea Vaccine Survey\HBK_Public\HBK_W3_20220926.csv", clear
gen WAVE = 3
save "HBK_W3", replace

import delimited "$rawdata\01_Survey\2022_Korea Vaccine Survey\HBK_Public\HBK_W4_20221128.csv", clear
gen WAVE = 4
save "HBK_W4", replace

use "HBK_W1", clear
append using HBK_W2, force
append using HBK_W3, force
append using HBK_W4, force
save "HBK_W1_W4", replace
```

    
    (encoding automatically selected: UTF-8)
    (165 vars, 1,500 obs)
    
    
    file HBK_W1.dta saved
    
    (encoding automatically selected: UTF-8)
    (150 vars, 1,500 obs)
    
    
    file HBK_W2.dta saved
    
    (encoding automatically selected: UTF-8)
    (143 vars, 1,500 obs)
    
    
    file HBK_W3.dta saved
    
    (encoding automatically selected: UTF-8)
    (222 vars, 1,500 obs)
    
    
    file HBK_W4.dta saved
    
    
    (note: variable rel_text was byte in the using data, but will be str23 now)
    
    (note: variable rel_text was byte in the using data, but will be str23 now)
    (variable vc002f_text was str20, now str32 to accommodate using data's values)
    (variable job_text was str6, now str18 to accommodate using data's values)
    
    (variable enddate was str15, now str16 to accommodate using data's values)
    
    file HBK_W1_W4.dta saved
    


```stata
use "HBK_W1_W4", clear
gen DATA = "Vaccine"

/*----- Variables -----*/
* Vaccination
recode vc001 (4=0 "Never") (3=1 "First") (2=2 "Second") (1=3 "Booster or more") if WAVE<=2, gen(VAX)
replace VAX = 0 if vc001==5 & WAVE>2
replace VAX = 1 if vc001==4 & WAVE>2
replace VAX = 2 if vc001==3 & WAVE>2
replace VAX = 3 if vc001==2 & WAVE>2
replace VAX = 4 if vc001==1 & WAVE>2
recode vc001 (4=0 "Never") (1/3=1 "1 or more") if WAVE<=2, gen(VAX1)
replace VAX1 = 0 if vc001==5 & WAVE>2
replace VAX1 = 1 if vc001>=1 & vc001<=4 & WAVE>2
recode vc001 (3/4=0 "<2") (1/2=1 "2 or more") if WAVE<=2, gen(VAX2)
replace VAX2 = 0 if vc001>=4 & vc001<=5 & WAVE>2
replace VAX2 = 1 if vc001>=1 & vc001<=3 & WAVE>2
recode vc001 (2/4=0 "<3") (1=1 "3 or more") if WAVE<=2, gen(VAX3)
replace VAX3 = 0 if vc001>=3 & vc001<=5 & WAVE>2
replace VAX3 = 1 if vc001>=1 & vc001<=2 & WAVE>2
replace VAX3 = 1 if VAX>=22 & vc002d==4
recode vc001 (2/5=0 "<4") (1=1 "4 or more") if WAVE>2, gen(VAX4)
recode vc004 (1/3=0 "No") (4/5=1 "Yes"), gen(VAX_ADD)

label var VAX "Vaccination"
label var VAX1 "1 Dose or More"
label var VAX2 "2 Doses or More"
label var VAX3 "Booster or More"
label var VAX_ADD "Will to Another"

* COVID-19 Risk Perception
recode hb001 (1/3=0 "No") (4 5=1 "Yes"), gen(COVID_EXGG)
label var COVID_EXGG "COVID-19 is exaggerated"
*label define COVID_EXGG 0 "No" 1 "Yes"
*label val COVID_EXGG COVID_EXGG

gen INFECTED = .
replace INFECTED = 0 if hl001a==1
replace INFECTED = 1 if hl001b==2
replace INFECTED = 2 if hl001c==3
replace INFECTED = 2 if hl001d==4
replace INFECTED = 2 if hl001e==5
replace INFECTED = 2 if hl001f==6
label var INFECTED "COVID-19 infection"
label define INFECTED 0 "Not infected" 1 "Myself was infected" 2 "Others were infected"
label val INFECTED INFECTED

* Remote work
recode pp005 (2 3=0 "No") (1 4=1 "Yes"), gen(RMTWRK)

* Trust
recode tr002b (1 2=0 "No") (3 4=1 "Yes"), gen(TR_KDCA)

* Politics
recode po001 (1 2=0 "Liberal") (3=2 "Independent") (4 5=1 "Conservative"), gen(IDEOLOGY)
label var IDEOLOGY "Ideology"

* Sociodemographic variables
recode age (1/29=0 "18-29") (30/39=1 "30s") (40/49=2 "40s") (50/59=3 "50s") (60/100=4 "60s"), gen(AGE)
recode sex (1=0 "Male") (2=1 "Female"), gen(FEMALE)
recode edu (1 2=0 "High school or less") (3=1 "Associate/Bachelor") (4 5=2 "Graduate"), gen(EDU)
recode job (2=0 "Employee") (1=1 "Health care worker") (3/7=2 "Other") if WAVE<=2, gen(JOB)
replace JOB = 0 if (job==7 | job==8) & WAVE>2
replace JOB = 1 if (job==1) & WAVE>2
replace JOB = 2 if (job>=2 & job<=6) & WAVE>2
recode job (4 5 6=1 "Unemployed") (1 2 3 7=0 "Other") if WAVE<=2, gen(UNEMPLOYED)
replace UNEMPLOYED = 1 if (job==3 | job==4 | job==5) & WAVE>2
replace UNEMPLOYED = 0 if (job==1 | job==2 | job==6 | job==7 | job==8) & WAVE>2
recode ms (2/4=0 "Not married") (1=1 "Married"), gen(MS)
recode rel (5=0 "No religion") (1/4 6=1 "Religion"), gen(REL)
recode income (1/4=0 "<₩2M") (5/8=1 "₩2M-₩3.99M") (9/10=2 "₩4M-₩5.99M") (11/12=3 "₩6M-₩7.99M") (13/15=4 ">₩7.99M"), gen(INCOME)

label var AGE "Age"
label var FEMALE "Female"
label var EDU "Education"
label var JOB "Job Status"
label var UNEMPLOYED "Unemployed"
label var MS "Marital Status"
label var REL "Religiosity"
label var INCOME "HH Income"

egen count_sample = count(DATA), by (EDU)

xtset prvc
save "HBK_W1_W4_clean", replace
```

    
    
    
    (2,167 differences between vc001 and VAX)
    
    (168 real changes made)
    
    (48 real changes made)
    
    (825 real changes made)
    
    (1,585 real changes made)
    
    (374 real changes made)
    
    (1,023 differences between vc001 and VAX1)
    
    (168 real changes made)
    
    (2,832 real changes made)
    
    (1,023 differences between vc001 and VAX2)
    
    (216 real changes made)
    
    (2,784 real changes made)
    
    (1,023 differences between vc001 and VAX3)
    
    (1,041 real changes made)
    
    (1,959 real changes made)
    
    (0 real changes made)
    
    (2,626 differences between vc001 and VAX4)
    
    (6,000 differences between vc004 and VAX_ADD)
    
    (6,000 differences between vc005a and VAX_EFF)
    
    (6,000 differences between vc005b and VAX_SIDE)
    
    (6,000 differences between vc005c and VAX_BNF)
    
    
    
    
    
    
    
    
    
    (1,500 differences between exp1_1 and EXP1_OUTCOME1)
    
    (1,500 differences between exp1_2 and EXP1_OUTCOME2)
    
    (4,500 missing values generated)
    
    (6,000 differences between hb002 and MASK_OUT)
    
    (6,000 differences between hb003 and MASK_IN)
    
    (6,000 differences between hb006 and MASK_EFF)
    
    (6,000 differences between hb001 and COVID_EXGG)
    
    
    (6,000 missing values generated)
    
    (752 real changes made)
    
    (2,094 real changes made)
    
    (3,644 real changes made)
    
    (518 real changes made)
    
    (487 real changes made)
    
    (131 real changes made)
    
    
    
    
    (5,646 differences between pp005 and RMTWRK)
    
    (6,000 differences between tr002a and TR_PRES)
    
    (6,000 differences between tr002b and TR_KDCA)
    
    (3,000 differences between tr002c and TR_PUB)
    
    (1,500 differences between tr002d and TR_DOC)
    
    
    
    
    
    (6,000 differences between tr003a and TR_CONMEDIA)
    
    (6,000 differences between tr003b and TR_LIBMEDIA)
    
    (6,000 differences between tr003c and TR_SOCMEDIA)
    
    (6,000 differences between tr004a and TR_FAM)
    
    (6,000 differences between tr004b and TR_FRNDS)
    
    (6,000 differences between pf001b and PF_EFFC)
    
    (6,000 differences between pf002b and PF_EQUI)
    
    (6,000 differences between pf003b and PF_TRAN)
    
    (3,000 differences between pf004b and PF_EFFI)
    
    (3,000 differences between pf005b and PF_SENS)
    
    
    
    
    
    
    (6,000 differences between po001 and IDEOLOGY)
    
    
    (3,766 differences between tr005a and anint1)
    
    (2,635 differences between tr005b and anint2)
    
    (3,465 differences between tr005c and anint3)
    
    (3,031 differences between tr005d and anint4)
    
    (2,703 differences between tr005e and anint5)
    
    (2,584 differences between tr005f and anint6)
    
    (2,571 differences between tr005g and anint7)
    
    (1,500 missing values generated)
    
    
    (6,000 differences between age and AGE)
    
    (6,000 differences between sex and FEMALE)
    
    (6,000 differences between edu and EDU)
    
    (2,831 differences between job and JOB)
    
    (1,193 real changes made)
    
    (568 real changes made)
    
    (1,239 real changes made)
    
    (3,000 differences between job and UNEMPLOYED)
    
    (895 real changes made)
    
    (2,105 real changes made)
    
    (2,798 differences between ms and MS)
    
    (4,724 differences between rel and REL)
    
    (6,000 differences between income and INCOME)
    
    
    
    
    
    
    
    
    
    
    
    Panel variable: prvc (unbalanced)
    
    file HBK_W1_W4_clean.dta saved
    

## Weighting


```stata
import delimited "PSweight_edu.csv", clear case(preserve)
capt drop count_sample pct_sample weight
save "PSweight_edu", replace
```

    
    (encoding automatically selected: UTF-8)
    (7 vars, 6 obs)
    
    
    file PSweight_edu.dta saved
    


```stata
* Weighting
merge m:1 EDU using "PSweight_edu.dta"
gen pct_sample = .
replace pct_sample = count_sample/6000
gen weight = pct_pop/pct_sample
gen Pweight = totalpop / sample
drop if _merge==2

save "_HBK_W1_W4_wt.dta", replace
```

# Part III. Data Review

### Weighted


```stata
capt prog drop ctab
program ctab, eclass
    syntax varlist [if] [in]
    // Row 1: No. of Observations
    qui tab WAVE
    matrix temp = r(N)
    local c1 = temp[1,1]
    qui tab WAVE if WAVE == 1
    matrix temp = r(N)
    local c2 = temp[1,1]
    qui tab WAVE if WAVE == 2
    matrix temp = r(N)
    local c3 = temp[1,1]
    qui tab WAVE if WAVE == 3
    matrix temp = r(N)
    local c4 = temp[1,1]
    qui tab WAVE if WAVE == 4
    matrix temp = r(N)
    local c5 = temp[1,1]
    mat M = (`c1',`c2',`c3',`c4',`c5', 0)
    mat rownames M = "No of Obs"
    mat colnames M = "All" "Wave 1" "Wave 2" "Wave 3" "Wave 4" "Chi-squared"

    local num = 0
    foreach x in `varlist'{
        local num = `num' + 1

        qui svy: mean `x'
        matrix temp = e(b)
        local m1 = temp[1,1]*100
        qui svy: mean `x' if WAVE==1
        matrix temp = e(b)
        local m2 = temp[1,1]*100
        qui svy: mean `x' if WAVE==2
        matrix temp = e(b)
        local m3 = temp[1,1]*100
        qui svy: mean `x' if WAVE==3
        matrix temp = e(b)
        local m4 = temp[1,1]*100
        qui svy: mean `x' if WAVE==4
        matrix temp = e(b)
        local m5 = temp[1,1]*100
        qui tab `x' WAVE, chi
        mat temp = r(p)
        local m6 = temp[1,1]

        mat temp = (`m1',`m2',`m3',`m4',`m5',`m6')
        local rowname: variable label `x'
        mat rownames temp = "`rowname'"
        mat M = M\temp
        // mat M = M[1..3,1...]
    }
    esttab matrix(M, fmt("0 2"))
    esttab matrix(M, fmt("0 2")) using "$outputs\Table1_weighted.rtf", replace
end
```


```stata
use "_HBK_W1_W4_wt", clear
drop if vc001==4 & WAVE<=2
drop if vc001==5 & WAVE>2
svyset [pweight=weight], strata(EDU)

qui tab IDEOLOGY, gen(IDEOLOGY_)
label var IDEOLOGY_1 "Liberal"
label var IDEOLOGY_2 "Independent"
label var IDEOLOGY_3 "Conservative"
qui tab AGE, gen(AGE_)
qui tab EDU, gen(EDU_)
label var EDU_1 "High school or less"
label var EDU_2 "BA"
label var EDU_3 "MA or more"
qui tab INCOME, gen(INCOME_)
label var INCOME_1 "<₩2M"
label var INCOME_2 "₩2M-₩3.99M"
label var INCOME_3 "₩4M-₩5.99M"
label var INCOME_4 "₩6M-₩7.99M"
label var INCOME_5 ">₩7.99M"

ctab VAX2 VAX3 VAX_ADD TR_KDCA IDEOLOGY_1-IDEOLOGY_3 COVID_EXGG FEMALE MS UNEMPLOYED REL ///
     AGE_1 AGE_2 AGE_3 AGE_4 AGE_5 EDU_1 EDU_2 EDU_3 INCOME_1 INCOME_2 INCOME_3 INCOME_4 INCOME_5
```

    
    
    (2,000 observations deleted)
    
    (148 observations deleted)
    
    (168 observations deleted)
    
    
    Sampling weights: weight
                 VCE: linearized
         Single unit: missing
            Strata 1: EDU
     Sampling unit 1: <observations>
               FPC 1: <zero>
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    ------------------------------------------------------------------------------------------
                            M                                                                 
                          All       Wave 1       Wave 2       Wave 3       Wave 4  Chi-squared
    ------------------------------------------------------------------------------------------
    No of Obs            5684         1423         1429         1421         1411            0
    2 Doses or~e        98.38        98.65        98.49        98.39        97.99         0.89
    Booster or~e        69.17        70.21        68.50        69.12        68.84         0.20
    Will to An~r        45.99        47.73        50.18        41.25        44.81         0.00
    Trust KDCA          70.02        70.91        77.42        68.14        63.70         0.00
    Liberal             24.17        23.33        26.48        22.86        24.03         0.34
    Independent         23.38        22.58        22.32        23.81        24.79         0.31
    Conservative        52.45        54.09        51.19        53.32        51.18         0.78
    COVID-19 i~d        35.55        35.95        34.34        36.47        35.42         0.59
    Female              51.28        50.93        51.44        51.52        51.25         0.99
    Marital St~s        52.90        52.19        55.36        51.34        52.75         0.63
    Unemployed          33.14        35.17        31.97        31.84        33.54         0.34
    Religiosity         46.84        47.16        47.42        46.22        46.57         0.71
    AGE==18-29          19.75        19.53        18.52        20.33        20.59         0.99
    AGE==30s            15.61        15.71        15.07        15.93        15.74         0.99
    AGE==40s            19.62        20.47        19.27        19.76        18.95         0.95
    AGE==50s            23.31        22.64        24.95        23.24        22.44         0.97
    AGE==60s            21.71        21.65        22.18        20.74        22.28         0.98
    High schoo~s        43.20        44.13        41.63        42.72        44.30         0.59
    BA                  50.63        49.69        51.77        51.54        49.54         0.62
    MA or more           6.17         6.18         6.59         5.74         6.16         0.64
    <₩2M                13.93        14.93        13.85        13.63        13.28         0.65
    ₩2M-₩3.99M          35.02        35.94        33.08        35.24        35.79         0.54
    ₩4M-₩5.99M          25.37        24.38        27.23        24.73        25.18         0.23
    ₩6M-₩7.99M          14.38        13.52        13.67        15.01        15.31         0.32
    >₩7.99M             11.31        11.23        12.16        11.40        10.44         0.92
    ------------------------------------------------------------------------------------------
    (output written to C:\Dropbox\01_Research\107_Korean Vaccine Survey\02_Analysis\outputs\01_Vaccination\Table1_weighted.rtf)
    

### Unweighted


```stata
capt prog drop ctab
program ctab, eclass
    syntax varlist [if] [in]
    // Row 1: No. of Observations
    qui tab WAVE
    matrix temp = r(N)
    local c1 = temp[1,1]
    qui tab WAVE if WAVE == 1
    matrix temp = r(N)
    local c2 = temp[1,1]
    qui tab WAVE if WAVE == 2
    matrix temp = r(N)
    local c3 = temp[1,1]
    qui tab WAVE if WAVE == 3
    matrix temp = r(N)
    local c4 = temp[1,1]
    qui tab WAVE if WAVE == 4
    matrix temp = r(N)
    local c5 = temp[1,1]
    mat M = (`c1',`c2',`c3',`c4',`c5', 0)
    mat rownames M = "No of Obs"
    mat colnames M = "All" "Wave 1" "Wave 2" "Wave 3" "Wave 4" "Chi-squared"

    local num = 0
    foreach x in `varlist'{
        local num = `num' + 1

        qui sum `x'
        matrix temp = r(mean)
        local m1 = temp[1,1]*100
        qui sum `x' if WAVE==1
        matrix temp = r(mean)
        local m2 = temp[1,1]*100
        qui sum `x' if WAVE==2
        matrix temp = r(mean)
        local m3 = temp[1,1]*100
        qui sum `x' if WAVE==3
        matrix temp = r(mean)
        local m4 = temp[1,1]*100
        qui sum `x' if WAVE==4
        matrix temp = r(mean)
        local m5 = temp[1,1]*100
        qui tab `x' WAVE, chi
        mat temp = r(p)
        local m6 = temp[1,1]

        mat temp = (`m1',`m2',`m3',`m4',`m5',`m6')
        local rowname: variable label `x'
        mat rownames temp = "`rowname'"
        mat M = M\temp
        // mat M = M[1..3,1...]
    }
    esttab matrix(M, fmt("0 2"))
    esttab matrix(M, fmt("0 2")) using "$outputs\eSupplement_unweighted.rtf", replace
end
```


```stata
use "_HBK_W1_W4_wt", clear
drop if vc001==4 & WAVE<=2
drop if vc001==5 & WAVE>2

qui tab IDEOLOGY, gen(IDEOLOGY_)
label var IDEOLOGY_1 "Liberal"
label var IDEOLOGY_2 "Independent"
label var IDEOLOGY_3 "Conservative"
qui tab AGE, gen(AGE_)
qui tab EDU, gen(EDU_)
label var EDU_1 "High school or less"
label var EDU_2 "BA"
label var EDU_3 "MA or more"
qui tab INCOME, gen(INCOME_)
label var INCOME_1 "<₩2M"
label var INCOME_2 "₩2M-₩3.99M"
label var INCOME_3 "₩4M-₩5.99M"
label var INCOME_4 "₩6M-₩7.99M"
label var INCOME_5 ">₩7.99M"

ctab VAX1 VAX2 VAX3 VAX4 VAX_ADD TR_KDCA IDEOLOGY_1-IDEOLOGY_3 COVID_EXGG FEMALE MS UNEMPLOYED REL AGE_1-AGE_5 EDU_1-EDU_3 INCOME_1-INCOME_5
```

    
    
    (148 observations deleted)
    
    (168 observations deleted)
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    ------------------------------------------------------------------------------------------
                            M                                                                 
                          All       Wave 1       Wave 2       Wave 3       Wave 4  Chi-squared
    ------------------------------------------------------------------------------------------
    No of Obs            5684         1423         1429         1421         1411            0
    1 Dose or ~e       100.00       100.00       100.00       100.00       100.00            .
    2 Doses or~e        98.42        98.59        98.46        98.38        98.23         0.89
    Booster or~e        69.25        71.19        67.46        69.25        69.10         0.20
    RECODE ~001)        13.21            .            .        11.40        15.02         0.00
    Will to An~r        45.43        46.66        49.27        41.45        44.29         0.00
    Trust KDCA          70.20        70.70        76.98        68.12        64.92         0.00
    Liberal             23.98        23.68        25.68        22.87        23.67         0.34
    Independent         23.49        22.98        22.04        24.14        24.81         0.31
    Conservative        52.53        53.34        52.27        52.99        51.52         0.78
    COVID-19 i~d        35.63        34.36        35.41        36.81        35.93         0.59
    Female              48.82        48.91        48.78        49.12        48.48         0.99
    Marital St~s        53.71        53.48        55.14        52.78        53.44         0.63
    Unemployed          29.87        31.76        28.97        29.28        29.48         0.34
    Religiosity         47.03        47.36        46.75        45.95        48.05         0.71
    AGE==18-29          19.65        19.61        19.59        19.49        19.91         0.99
    AGE==30s            17.31        17.50        17.14        17.10        17.51         0.99
    AGE==40s            21.06        21.43        21.13        21.11        20.55         0.95
    AGE==50s            22.68        22.42        22.95        22.94        22.40         0.97
    AGE==60s            19.30        19.04        19.17        19.35        19.63         0.98
    High schoo~s        22.27        22.91        21.13        22.03        23.03         0.59
    BA                  64.57        63.81        65.01        65.73        63.71         0.62
    MA or more          13.16        13.28        13.86        12.24        13.25         0.64
    <₩2M                11.56        12.37        11.55        11.47        10.84         0.65
    ₩2M-₩3.99M          33.15        33.45        31.63        33.43        34.09         0.54
    ₩4M-₩5.99M          25.95        25.58        27.99        25.26        24.95         0.23
    ₩6M-₩7.99M          16.06        15.32        15.05        16.68        17.22         0.32
    >₩7.99M             13.28        13.28        13.79        13.16        12.90         0.92
    ------------------------------------------------------------------------------------------
    (file C:\Dropbox\01_Research\107_Korean Vaccine Survey\02_Analysis\outputs\01_Vaccination\eSupplement_unweighted.rtf not found)
    (output written to C:\Dropbox\01_Research\107_Korean Vaccine Survey\02_Analysis\outputs\01_Vaccination\eSupplement_unweighted.rtf)
    

## Part IV. Estimation

## Figure 1 & Appendix 1. Predictors of COVID-19 Booster Vaccination


```stata
use "_HBK_W1_W4_wt", clear
qui drop if vc001==4 & WAVE<=2
qui drop if vc001==5 & WAVE>2
qui svyset [pweight=weight], strata(EDU)

loc controls i.COVID_EXGG i.FEMALE i.MS i.UNEMPLOYED i.REL i.IDEOLOGY i.AGE i.EDU i.INCOME i.WAVE
loc title1 "A. Booster Vaccination"
loc title2 "B. Willingness to Get Another (After the 2nd)"
loc title3 "C. Willingness to Get Another (After the 3rd)"
loc xrange -0.2(0.1)0.5

eststo lm1: qui xtreg VAX3 i.TR_KDCA `controls' [pw=Pweight], fe vce(cluster prvc)
qui coefplot (lm1, label(Logit) mfcolor(navy*0.8)), bylabel("`title1'") || ///
           , drop(*.WAVE _cons) ///
             headings(1.VAX3="{bf:Booster & trust}" 0.TR_KDCA="{bf:Trust in the KDCA}" 1.IDEOLOGY = "{bf:Ideology (ref. Liberal)}" ///
                 1.COVID_EXGG = " " 0.FEMALE = "{bf:Gender}" ///
                 1.AGE = "{bf:Age (ref. 18-29)}" 0.MS = "{bf:Marital Status}" 0.UNEMPLOYED = "{bf:Unemployed}" 0.REL = "{bf:Having a Religion}" ///
                 1.EDU = "{bf:Education (ref. <Associate)}" ///
                 1.INCOME = "{bf:Income (ref. <₩2M)}") ///
             coeflabels(1.TR_KDCA="Trust KDCA" ///
                        1.COVID_EXGG="Risk perception" 1.FEMALE="Female" 1.MS="Married" 1.UNEMPLOYED="Unemployed" 1.REL="Having religion" ///
                        1.IDEOLOGY="Conservative" 2.IDEOLOGY="Independent") ///
             xtitle("{bf:Point Estimates} (OLS Coefficients)", margin(small)) ytitle({bf:Variable}, margin(small)) ///
             xlabel(`xrange', grid glcolor(gs4)) ylabel(, nogrid) byopts(row(1)) ///
             mlabel("     " + string(@b, "%9.2f") + " [" + string(@ll, "%9.2f") + "," + string(@ul, "%9.2f") + "]") mlabformat("%9.2f") ///
             xline(0, lcolor(gray) lwidth(medium)) ///
             plotregion(lwidth(thin) lpattern(solid)) ///
             msize(medium) mlwidth(vthin) msymbol(S) mfcolor(`r(p1)'*0.8) mlcolor(black) ///
             ciopts(recast(rspike) lcolor(black) lwidth(vthin)) mlabel format(%9.2f) mlabcolor(black) mlabposition(3) mlabgap(*16) ///
             graphregion(fcolor(white) color(white) icolor(white) margin(small)) ///
             legend(off) ///
             name(g1, replace)

qui graph combine g1, ///
        b1("") ycommon xcommon ///
        l1("") ///
        xsize(12) ysize(12) iscale(*0.55)
qui graph export "$outputs\Fig1.jpg", replace
```


```stata
esttab lm1  ///
    ,replace b(4) ci(2) r2(2) ar2(2) scalar(F) ///
    title(Figure 1) nogaps ///
    varwidth(18) modelwidth(15) nobase label
```

    
    Figure 1
    -------------------------------------
                                   (1)   
                       Booster or More   
    -------------------------------------
    Yes                         0.0833***
                           [0.06,0.11]   
    Yes                        -0.0178   
                          [-0.05,0.01]   
    Female                     -0.0387** 
                         [-0.06,-0.01]   
    Married                     0.0067   
                          [-0.02,0.03]   
    Unemployed                 -0.0202   
                          [-0.06,0.02]   
    Religion                    0.0196   
                          [-0.00,0.04]   
    Conservative               -0.0185   
                          [-0.05,0.02]   
    Independent                -0.0332*  
                         [-0.07,-0.00]   
    30s                        -0.0379   
                          [-0.09,0.01]   
    40s                         0.0569*  
                           [0.00,0.11]   
    50s                         0.1724***
                           [0.13,0.22]   
    60s                         0.2561***
                           [0.21,0.31]   
    Associate/Bachelor          0.0277   
                          [-0.00,0.06]   
    Graduate                    0.0296   
                          [-0.01,0.07]   
    ₩2M-₩3.99M                  0.0507** 
                           [0.02,0.08]   
    ₩4M-₩5.99M                  0.0194   
                          [-0.05,0.09]   
    ₩6M-₩7.99M                  0.0345   
                          [-0.03,0.10]   
    >₩7.99M                     0.0455   
                          [-0.02,0.11]   
    WAVE=2                     -0.0457** 
                         [-0.08,-0.01]   
    WAVE=3                     -0.0197   
                          [-0.05,0.01]   
    WAVE=4                     -0.0183   
                          [-0.05,0.02]   
    Constant                    0.5460***
                           [0.49,0.60]   
    -------------------------------------
    Observations                  5684   
    R-squared                     0.07   
    Adjusted R-squared            0.07   
    F                                .   
    -------------------------------------
    95% confidence intervals in brackets
    * p<0.05, ** p<0.01, *** p<0.001
    

![](img\Fig1.jpg)

## Appendix 2. Unadjusted Estimates of the Predictors of COVID-19 Vaccination


```stata
use "_HBK_W1_W4_wt", clear
qui drop if vc001==4 & WAVE<=2
qui drop if vc001==5 & WAVE>2
qui svyset [pweight=weight], strata(EDU)

loc vars i.TR_KDCA i.IDEOLOGY i.COVID_EXGG i.FEMALE i.MS i.UNEMPLOYED i.REL i.AGE i.EDU i.INCOME
loc title1 "A. Booster Vaccination"
loc title2 "B. Willingness to Get Another (After the 2nd)"
loc xrange -0.4(0.1)0.5

matrix M = J(28, 3, .)
matrix coln M = Beta LI95 UI95

qui xtreg VAX3 i.TR_KDCA i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[1,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[2,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX3 i.IDEOLOGY i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[3,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[4,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[5,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
qui xtreg VAX3 i.COVID_EXGG i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[6,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[7,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX3 i.FEMALE i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[8,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[9,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX3 i.MS i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[10,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[11,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX3 i.UNEMPLOYED i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[12,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[13,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX3 i.REL i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[14,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[15,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX3 i.AGE i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[16,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[17,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[18,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
matrix M[19,1] = r(table)[1,4], r(table)[5,4], r(table)[6,4]
matrix M[20,1] = r(table)[1,5], r(table)[5,5], r(table)[6,5]
qui xtreg VAX3 i.EDU i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[21,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[22,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[23,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
qui xtreg VAX3 i.INCOME i.WAVE [pw=Pweight], fe vce(cluster prvc)
matrix M[24,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[25,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[26,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
matrix M[27,1] = r(table)[1,4], r(table)[5,4], r(table)[6,4]
matrix M[28,1] = r(table)[1,5], r(table)[5,5], r(table)[6,5]

qui coefplot (matrix(M[,1]), label(Wave 1) ci((2 3)) msymbol(S) mfcolor(navy*0.8)) ///
     , xtitle(, margin(small)) ///
       ytitle("Marginal Effect", margin(tiny)) ///
       msymbol(s) byopts(cols(6)) ///
       headings(r1="{bf:Trust in the KDCA}" r3 = "{bf:Ideology}" ///
                 r6 = "{bf:COVID-19 is exaggerated}" r8 = "{bf:Gender}" ///
                 r10 = "{bf:Marital Status}" r12 = "{bf:Unemployed}" r14 = "{bf:Having a Religion}" ///
                 r16 = "{bf:Age}" r21 = "{bf:Education}" r24 = "{bf:Income}") ///       
       coeflabels(r1 = "No" r2 = "Yes" r3 = "Liberal" r4 = "Conservative" r5 = "Independent" ///
                  r6 = "No" r7 = "Yes" r8 = "Male" r9 = "Female" r10 = "Not married" r11 = "Married" ///
                  r12 = "No" r13 = "Yes" r14 = "No" r15 = "Yes" ///
                  r16 = "18-29" r17 = "30s" r18 = "40s" r19 = "50s" r20 = "60s" ///
                  r21 = "High school or less" r22 = "Associate/Bachelor" r23 = "Graduate" ///
                  r24 = "<₩2M" r25 = "₩2M-₩3.99M" r26 = "₩4M-₩5.99M" r27 = "₩6M-₩7.99M" r28 = ">₩7.99M") ///       
       xtitle("{bf:Point Estimate} (OLS Coefficient)", margin(small)) ytitle({bf:Variables}, margin(small)) ///
       xlabel(`xrange', grid glcolor(gs4)) ylabel(, nogrid) ///
       xline(0, lcolor(gray) lwidth(medium)) ///
       plotregion(lwidth(thin) lpattern(solid)) ///
       msize(medium) mlwidth(vthin) msymbol(S) mfcolor(`r(p1)'*0.8) mlcolor(black) ///
       ciopts(recast(rspike) lcolor(black) lwidth(vthin)) mlabel format(%9.2f) mlabcolor(black) mlabposition(3) mlabgap(*12) ///
       graphregion(fcolor(white) color(white) icolor(white) margin(small)) ///
       legend(off) ///
       name(g1, replace)
qui graph combine g1, ///
    b1("") ycommon xcommon ///
    l1("") ///
    xsize(12) ysize(12) iscale(*0.55)
qui graph export "$outputs\Appendix2.jpg", replace
```

![](img\Appendix2.jpg)

## Appendix 3. Predictors of COVID-19 Vaccination (Logistic Regression)


```stata
use "_HBK_W1_W4_wt", clear
qui drop if vc001==4 & WAVE<=2
qui drop if vc001==5 & WAVE>2
qui svyset [pweight=weight], strata(EDU)

loc controls i.COVID_EXGG i.FEMALE i.MS i.UNEMPLOYED i.REL i.AGE i.EDU i.INCOME i.WAVE
loc title1 "A. Booster Vaccination"
loc title2 "B. Willingness to Get Another (After the 2nd)"
loc title3 "C. Willingness to Get Another (After the 3rd)"
loc xrange 0(1)6

eststo lm1: qui xtlogit VAX3 i.TR_KDCA i.IDEOLOGY `controls', or vce(cluster prvc)
qui coefplot (lm1, label(Logit) mfcolor(navy*0.8)), bylabel("`title1'") || ///
           , eform drop(*.WAVE _cons) ///
             headings(1.VAX3="{bf:Booster & trust}" 0.TR_KDCA="{bf:Trust in the KDCA}" 1.IDEOLOGY = "{bf:Ideology (ref. Liberal)}" ///
                 1.COVID_EXGG = " " 0.FEMALE = "{bf:Gender}" ///
                 1.AGE = "{bf:Age (ref. 18-29)}" 0.MS = "{bf:Marital Status}" 0.UNEMPLOYED = "{bf:Unemployed}" 0.REL = "{bf:Having a Religion}" ///
                 1.EDU = "{bf:Education (ref. <Associate)}" ///
                 1.INCOME = "{bf:Income (ref. <₩2M)}") ///
             coeflabels(1.TR_KDCA="Trust KDCA" ///
                        1.COVID_EXGG="Risk perception" 1.FEMALE="Female" 1.MS="Married" 1.UNEMPLOYED="Unemployed" 1.REL="Having religion" ///
                        1.IDEOLOGY="Conservative" 2.IDEOLOGY="Independent") ///
             xtitle("{bf:Point Estimate} (Odds Ratio)", margin(small)) ytitle({bf:Variables}, margin(small)) ///
             xlabel(`xrange', grid glcolor(gs4)) ylabel(, nogrid) byopts(row(1)) ///
             xline(1, lcolor(gray) lwidth(medium)) ///
             plotregion(lwidth(thin) lpattern(solid)) ///
             msize(medium) mlwidth(vthin) msymbol(S) mfcolor(`r(p1)'*0.8) mlcolor(black) ///
             ciopts(recast(rspike) lcolor(black) lwidth(vthin)) mlabel format(%9.2f) mlabcolor(black) mlabposition(3) mlabgap(*12) ///
             graphregion(fcolor(white) color(white) icolor(white) margin(small)) ///
             legend(off) ///
             name(g1, replace)

qui graph combine g1, ///
        b1("") ycommon xcommon ///
        l1("") ///
        xsize(12) ysize(12) iscale(*0.55)
qui graph export "$outputs\Appendix3.jpg", replace
```


```stata
esttab lm1  ///
    , eform replace b(4) ci(2) r2(2) ar2(2) scalar(F) ///
    title(Appendix 3) nogaps ///
    varwidth(18) modelwidth(15) nobase label
```

    
    Appendix 3
    -------------------------------------
                                   (1)   
                       Booster or More   
    -------------------------------------
    Booster or More                      
    Yes                         1.5101***
                           [1.36,1.68]   
    Conservative                0.9031   
                           [0.76,1.07]   
    Independent                 0.8352*  
                           [0.72,0.97]   
    Yes                         0.9145   
                           [0.80,1.05]   
    Female                      0.8119***
                           [0.72,0.92]   
    Married                     1.0387   
                           [0.92,1.17]   
    Unemployed                  0.8954   
                           [0.75,1.08]   
    Religion                    1.1029   
                           [0.99,1.23]   
    30s                         0.8415   
                           [0.69,1.02]   
    40s                         1.2625*  
                           [1.01,1.58]   
    50s                         2.2749***
                           [1.84,2.81]   
    60s                         4.0218***
                           [2.96,5.47]   
    Associate/Bachelor          1.1518   
                           [1.00,1.33]   
    Graduate                    1.1703   
                           [0.94,1.45]   
    ₩2M-₩3.99M                  1.2729***
                           [1.10,1.47]   
    ₩4M-₩5.99M                  1.0904   
                           [0.79,1.51]   
    ₩6M-₩7.99M                  1.1691   
                           [0.87,1.57]   
    >₩7.99M                     1.2438   
                           [0.94,1.65]   
    WAVE=2                      0.7942** 
                           [0.68,0.93]   
    WAVE=3                      0.9045   
                           [0.78,1.05]   
    WAVE=4                      0.9071   
                           [0.77,1.07]   
    -------------------------------------
    /                                    
    lnsig2u                     0.0165***
                           [0.00,0.09]   
    -------------------------------------
    Observations                  5684   
    R-squared                            
    Adjusted R-squared                   
    F                                    
    -------------------------------------
    Exponentiated coefficients; 95% confidence intervals in brackets
    * p<0.05, ** p<0.01, *** p<0.001
    

![](img\Appendix3.jpg)

## Figure 2 & Appendix 4. Predictors of the Willingness to Get an Additional Vaccine Dose Against COVID-19


```stata
use "_HBK_W1_W4_wt", clear
qui drop if vc001==4 & WAVE<=2
qui drop if vc001==5 & WAVE>2
qui svyset [pweight=weight], strata(EDU)

loc controls i.COVID_EXGG i.FEMALE i.MS i.UNEMPLOYED i.REL i.IDEOLOGY i.AGE i.EDU i.INCOME i.WAVE
loc title2 "A. Willingness to Get Another (After the 1st or the 2nd Shot, n = 1,748)"
loc title3 "B. Willingness to Get Another (After the 3rd Shot, n = 3,562)"
loc xrange -0.2(0.1)0.5

eststo lm2: qui xtreg VAX_ADD i.VAX3##i.TR_KDCA `controls' [pw=Pweight], fe vce(cluster prvc)
qui margins, dydx(TR_KDCA)
qui margins VAX3, dydx(TR_KDCA)
qui margins TR_KDCA, dydx(VAX3)

qui coefplot (lm2, label(Logit) mfcolor(navy*0.8)) || ///
           , drop(*.WAVE _cons) ///
             headings(1.VAX3="{bf:Booster & trust}" 0.TR_KDCA="{bf:Trust in the KDCA}" 1.IDEOLOGY = "{bf:Ideology}" ///
                 1.COVID_EXGG = " " 0.FEMALE = "{bf:Gender}" ///
                 1.AGE = "{bf:Age}" 0.MS = "{bf:Marital Status}" 0.UNEMPLOYED = "{bf:Unemployed}" 0.REL = "{bf:Having a Religion}" 1.EDU = "{bf:Education}" ///
                 1.INCOME = "{bf:Income}") ///
             coeflabels(1.VAX3="Booster shot (low trust)" 1.TR_KDCA="Trust KDCA (1-2 doses)" 1.VAX3#1.TR_KDCA="Interaction" ///
                        1.COVID_EXGG="Risk perception" 1.FEMALE="Female" 1.MS="Married" 1.UNEMPLOYED="Unemployed" 1.REL="Having religion" ///
                        1.IDEOLOGY="Conservative" 2.IDEOLOGY="Independent") ///
             xtitle("{bf:Point Estimates} (OLS Coefficients)", margin(small)) ytitle({bf:Variables}, margin(small)) ///
             xlabel(`xrange', grid glcolor(gs4)) ylabel(, nogrid) byopts(row(1)) ///
             mlabel("     " + string(@b, "%9.2f") + " [" + string(@ll, "%9.2f") + "," + string(@ul, "%9.2f") + "]") mlabformat("%9.2f") ///
             xline(0, lcolor(gray) lwidth(medium)) ///
             plotregion(lwidth(thin) lpattern(solid)) ///
             msize(medium) mlwidth(vthin) msymbol(S) mfcolor(`r(p1)'*0.8) mlcolor(black) ///
             ciopts(recast(rspike) lcolor(black) lwidth(vthin)) mlabel format(%9.2f) mlabcolor(black) mlabposition(3) mlabgap(15) ///
             graphregion(fcolor(white) color(white) icolor(white) margin(small)) ///
             legend(off) ///
             name(g1, replace)

qui graph combine g1, ///
        b1("") ycommon xcommon ///
        l1("") ///
        xsize(12) ysize(12) iscale(*0.55)
qui graph export "$outputs\Fig2.jpg", replace
```


```stata
esttab lm2 ///
    ,replace b(4) ci(2) r2(2) ar2(2) scalar(F) ///
    title(Figure 2) nogaps ///
    varwidth(18) modelwidth(15) nobase label
```

    
    Figure 2
    -------------------------------------
                                   (1)   
                       Will to Another   
    -------------------------------------
    3 or more                   0.1777***
                           [0.15,0.21]   
    Yes                         0.1376***
                           [0.09,0.18]   
    3 or more # Yes             0.0683*  
                           [0.01,0.13]   
    Yes                         0.0243   
                          [-0.01,0.06]   
    Female                     -0.0834***
                         [-0.11,-0.06]   
    Married                    -0.0290   
                          [-0.06,0.01]   
    Unemployed                 -0.0249   
                          [-0.06,0.01]   
    Religion                    0.0086   
                          [-0.02,0.04]   
    Conservative               -0.0393   
                          [-0.08,0.00]   
    Independent                -0.0718** 
                         [-0.11,-0.03]   
    30s                         0.0574** 
                           [0.02,0.10]   
    40s                         0.1249***
                           [0.08,0.17]   
    50s                         0.1369***
                           [0.11,0.17]   
    60s                         0.1929***
                           [0.16,0.23]   
    Associate/Bachelor         -0.0357   
                          [-0.08,0.00]   
    Graduate                   -0.0179   
                          [-0.07,0.03]   
    ₩2M-₩3.99M                 -0.0315   
                          [-0.07,0.01]   
    ₩4M-₩5.99M                 -0.0118   
                          [-0.07,0.04]   
    ₩6M-₩7.99M                  0.0101   
                          [-0.02,0.04]   
    >₩7.99M                     0.0144   
                          [-0.03,0.06]   
    WAVE=2                      0.0214   
                          [-0.02,0.06]   
    WAVE=3                     -0.0441   
                          [-0.10,0.01]   
    WAVE=4                     -0.0086   
                          [-0.05,0.03]   
    Constant                    0.2367***
                           [0.16,0.31]   
    -------------------------------------
    Observations                  5684   
    R-squared                     0.13   
    Adjusted R-squared            0.12   
    F                                .   
    -------------------------------------
    95% confidence intervals in brackets
    * p<0.05, ** p<0.01, *** p<0.001
    

![](img\Fig2.jpg)

## Appendix 5. Unadjusted Estimates of the Willingness to Get an Additional Vaccine Dose Against COVID-19


```stata
use "_HBK_W1_W4_wt", clear
qui drop if vc001==4 & WAVE<=2
qui drop if vc001==5 & WAVE>2

loc vars i.TR_KDCA i.IDEOLOGY i.COVID_EXGG i.FEMALE i.MS i.UNEMPLOYED i.REL i.AGE i.EDU i.INCOME
loc title1 "A. Booster Vaccination"
loc title2 "B. Willingness to Get Another (After the 2nd)"
loc xrange -0.4(0.1)0.4

matrix M = J(28, 3, .)
matrix coln M = Beta LI95 UI95

qui xtreg VAX_ADD i.TR_KDCA i.WAVE, fe vce(cluster prvc)
matrix M[1,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[2,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX_ADD i.IDEOLOGY i.WAVE, fe vce(cluster prvc)
matrix M[3,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[4,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[5,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
qui xtreg VAX_ADD i.COVID_EXGG i.WAVE, fe vce(cluster prvc)
matrix M[6,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[7,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX_ADD i.FEMALE i.WAVE, fe vce(cluster prvc)
matrix M[8,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[9,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX_ADD i.MS i.WAVE, fe vce(cluster prvc)
matrix M[10,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[11,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX_ADD i.UNEMPLOYED i.WAVE, fe vce(cluster prvc)
matrix M[12,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[13,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX_ADD i.REL i.WAVE, fe vce(cluster prvc)
matrix M[14,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[15,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
qui xtreg VAX_ADD i.AGE i.WAVE, fe vce(cluster prvc)
matrix M[16,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[17,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[18,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
matrix M[19,1] = r(table)[1,4], r(table)[5,4], r(table)[6,4]
matrix M[20,1] = r(table)[1,5], r(table)[5,5], r(table)[6,5]
qui xtreg VAX_ADD i.EDU i.WAVE, fe vce(cluster prvc)
matrix M[21,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[22,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[23,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
qui xtreg VAX_ADD i.INCOME i.WAVE, fe vce(cluster prvc)
matrix M[24,1] = r(table)[1,1], r(table)[5,1], r(table)[6,1]
matrix M[25,1] = r(table)[1,2], r(table)[5,2], r(table)[6,2]
matrix M[26,1] = r(table)[1,3], r(table)[5,3], r(table)[6,3]
matrix M[27,1] = r(table)[1,4], r(table)[5,4], r(table)[6,4]
matrix M[28,1] = r(table)[1,5], r(table)[5,5], r(table)[6,5]

qui coefplot (matrix(M[,1]), label(Wave 1) ci((2 3)) msymbol(S) mfcolor(navy*0.8)) ///
     , xtitle(, margin(small)) ///
       ytitle("Marginal Effect", margin(tiny)) ///
       msymbol(s) byopts(cols(6)) ///
       headings(r1="{bf:Trust in the KDCA}" r3 = "{bf:Ideology}" ///
                 r6 = "{bf:COVID-19 is exaggerated}" r8 = "{bf:Gender}" ///
                 r10 = "{bf:Marital Status}" r12 = "{bf:Unemployed}" r14 = "{bf:Having a Religion}" ///
                 r16 = "{bf:Age}" r21 = "{bf:Education}" r24 = "{bf:Income}") ///       
       coeflabels(r1 = "No" r2 = "Yes" r3 = "Liberal" r4 = "Conservative" r5 = "Independent" ///
                  r6 = "No" r7 = "Yes" r8 = "Male" r9 = "Female" r10 = "Not married" r11 = "Married" ///
                  r12 = "No" r13 = "Yes" r14 = "No" r15 = "Yes" ///
                  r16 = "18-29" r17 = "30s" r18 = "40s" r19 = "50s" r20 = "60s" ///
                  r21 = "High school or less" r22 = "Associate/Bachelor" r23 = "Graduate" ///
                  r24 = "<₩2M" r25 = "₩2M-₩3.99M" r26 = "₩4M-₩5.99M" r27 = "₩6M-₩7.99M" r28 = ">₩7.99M") ///
       xtitle("{bf:Point Estimate} (OLS Coefficient)", margin(small)) ytitle({bf:Variables}, margin(small)) ///
       xlabel(`xrange', grid glcolor(gs4)) ylabel(, nogrid) ///
       xline(0, lcolor(gray) lwidth(medium)) ///
       plotregion(lwidth(thin) lpattern(solid)) ///
       msize(medium) mlwidth(vthin) msymbol(S) mfcolor(`r(p1)'*0.8) mlcolor(black) ///
       ciopts(recast(rspike) lcolor(black) lwidth(vthin)) mlabel format(%9.2f) mlabcolor(black) mlabposition(3) mlabgap(*12) ///
       graphregion(fcolor(white) color(white) icolor(white) margin(small)) ///
       legend(off) ///
       name(g1, replace)
qui graph combine g1, ///
    b1("") ycommon xcommon ///
    l1("") ///
    xsize(12) ysize(12) iscale(*0.55)
qui graph export "$outputs\Appendix5.jpg", replace
```

![](img\Appendix5.jpg)

## Appendix 6. Predictors of the Willingness to Get an Additional Vaccine Dose Against COVID-19 (Logistic Regression)


```stata
use "_HBK_W1_W4_wt", clear
qui drop if vc001==4 & WAVE<=2
qui drop if vc001==5 & WAVE>2
qui svyset [pweight=weight], strata(EDU)

loc controls i.COVID_EXGG i.FEMALE i.MS i.UNEMPLOYED i.REL i.AGE i.EDU i.INCOME i.WAVE
loc title2 "A. Willingness to Get Another (After the 1st or the 2nd Shot, n = 1,748)"
loc title3 "B. Willingness to Get Another (After the 3rd Shot, n = 3,562)"
loc xrange 0(1)6

eststo lm2: qui xtlogit VAX_ADD i.VAX3##i.TR_KDCA i.IDEOLOGY `controls', or vce(cluster prvc)
*eststo lm3: qui xtlogit VAX_ADD i.TR_KDCA i.IDEOLOGY `controls' if VAX==3, or vce(cluster prvc)
qui coefplot (lm2, label(Logit) mfcolor(navy*0.8)) || ///
           , eform drop(*.WAVE _cons) ///
             headings(1.VAX3="{bf:Booster & trust}" 0.TR_KDCA="{bf:Trust in the KDCA}" 1.IDEOLOGY = "{bf:Ideology}" ///
                 1.COVID_EXGG = " " 0.FEMALE = "{bf:Gender}" ///
                 1.AGE = "{bf:Age}" 0.MS = "{bf:Marital Status}" 0.UNEMPLOYED = "{bf:Unemployed}" 0.REL = "{bf:Having a Religion}" 1.EDU = "{bf:Education}" ///
                 1.INCOME = "{bf:Income}") ///
             coeflabels(1.VAX3="Booster shot (low trust)" 1.TR_KDCA="Trust KDCA (1-2 doses)" 1.VAX3#1.TR_KDCA="Interaction" ///
                        1.COVID_EXGG="Risk perception" 1.FEMALE="Female" 1.MS="Married" 1.UNEMPLOYED="Unemployed" 1.REL="Having religion" ///
                        1.IDEOLOGY="Conservative" 2.IDEOLOGY="Independent") ///
             xtitle("{bf:Point Estimate} (Odds ratio)", margin(small)) ytitle({bf:Variables}, margin(small)) ///
             xlabel(`xrange', grid glcolor(gs4)) ylabel(, nogrid) byopts(row(1)) ///
             xline(1, lcolor(gray) lwidth(medium)) ///
             plotregion(lwidth(thin) lpattern(solid)) ///
             msize(medium) mlwidth(vthin) msymbol(S) mfcolor(`r(p1)'*0.8) mlcolor(black) ///
             ciopts(recast(rspike) lcolor(black) lwidth(vthin)) mlabel format(%9.2f) mlabcolor(black) mlabposition(3) mlabgap(15) ///
             graphregion(fcolor(white) color(white) icolor(white) margin(small)) ///
             legend(off) ///
             name(g1, replace)

qui graph combine g1, ///
        b1("") ycommon xcommon ///
        l1("") ///
        xsize(12) ysize(12) iscale(*0.55)
qui graph export "$outputs\Appendix6.jpg", replace
```


```stata
esttab lm2 ///
    , eform replace b(4) ci(2) r2(2) ar2(2) scalar(F) ///
    title(Appendix 6) nogaps ///
    varwidth(18) modelwidth(15) nobase label
```

    
    Appendix 6
    -------------------------------------
                                   (1)   
                       Will to Another   
    -------------------------------------
    Will to Another                      
    3 or more                   2.6594***
                           [2.31,3.06]   
    Yes                         2.2019***
                           [1.79,2.71]   
    3 or more # Yes             1.0827   
                           [0.83,1.41]   
    Conservative                0.8275*  
                           [0.70,0.98]   
    Independent                 0.7146***
                           [0.61,0.84]   
    Yes                         1.1204   
                           [0.97,1.29]   
    Female                      0.6799***
                           [0.62,0.75]   
    Married                     0.8675   
                           [0.75,1.00]   
    Unemployed                  0.8948   
                           [0.75,1.06]   
    Religion                    1.0368   
                           [0.92,1.17]   
    30s                         1.3156** 
                           [1.10,1.58]   
    40s                         1.8057***
                           [1.51,2.16]   
    50s                         1.8880***
                           [1.65,2.16]   
    60s                         2.4163***
                           [2.07,2.82]   
    Associate/Bachelor          0.8398*  
                           [0.71,1.00]   
    Graduate                    0.9176   
                           [0.75,1.12]   
    ₩2M-₩3.99M                  0.8735   
                           [0.73,1.04]   
    ₩4M-₩5.99M                  0.9550   
                           [0.75,1.21]   
    ₩6M-₩7.99M                  1.0523   
                           [0.92,1.21]   
    >₩7.99M                     1.0748   
                           [0.89,1.30]   
    WAVE=2                      1.1017   
                           [0.93,1.30]   
    WAVE=3                      0.8174   
                           [0.64,1.04]   
    WAVE=4                      0.9619   
                           [0.82,1.13]   
    -------------------------------------
    /                                    
    lnsig2u                     0.0151***
                           [0.00,0.17]   
    -------------------------------------
    Observations                  5684   
    R-squared                            
    Adjusted R-squared                   
    F                                    
    -------------------------------------
    Exponentiated coefficients; 95% confidence intervals in brackets
    * p<0.05, ** p<0.01, *** p<0.001
    

![](img\Appendix6.jpg)
