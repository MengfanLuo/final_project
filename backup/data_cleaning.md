Data cleaning
================
Mengfan Luo
11/11/2021

### Sample code given with the source data

    rm(list=ls(all=TRUE))
    library("haven")
    library("survey")
    library(dplyr)
    require(data.table)
    chs17<-read_sas("data/chs2017_public.sas7bdat")

    #city-wide estimates
    chs<-transform(chs17,strata=as.character(strata),all=as.factor(survey))

    #define the survey
    chs.dsgn<-svydesign(ids = ~1,strata = ~strata,weights=~wt18_dual,data = chs,nest = TRUE,na.rm=TRUE )
    #age adjusted survey
    pop.agecat4=c(0.128810, 0.401725, 0.299194, 0.170271)
    chs.stdes<-svystandardize(subset(chs.dsgn,diabetes17>0 ),by=~agegroup,over=~all,population=pop.agecat4,excluding.missing =~ agegroup+ ~all)

    #weighted N
    aggregate(chs17$wt18_dual, by=list(Category=chs17$diabetes17), FUN=sum)

    #crude prevalance estimates
    svyby(~diabetes17==1,~all,subset(chs.dsgn,diabetes17>0),svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))
    svyby(~diabetes17==2,~all,subset(chs.dsgn,diabetes17>0),svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))

    #age adjusted prevalance estimates

    svyby(~diabetes17==1,~all,chs.stdes,svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))
    svyby(~diabetes17==2,~all,chs.stdes,svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))

    #estimate by sex
    chs<-transform(chs17,strata=as.character(strata),allsex2=as.factor(sex))

    #define the survey
    chs.dsgn<-svydesign(ids = ~1,strata = ~strata,weights=~wt18_dual,data = chs,nest = TRUE,na.rm=TRUE )
    #age adjusted survey
    pop.agecat4=c(0.128810, 0.401725, 0.299194, 0.170271)
    chs.stdes<-svystandardize(subset(chs.dsgn,diabetes17>0 ),by=~agegroup,over=~allsex2,population=pop.agecat4,excluding.missing =~ agegroup+ ~allsex2)

    #crude prevalance estimates
    svyby(~diabetes17==1,~allsex2,subset(chs.dsgn,diabetes17>0),svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))
    svyby(~diabetes17==2,~allsex2,subset(chs.dsgn,diabetes17>0),svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))


    #age adjusted prevalance estimates

    svyby(~diabetes17==1,~allsex2,chs.stdes,svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))
    svyby(~diabetes17==2,~allsex2,chs.stdes,svyciprop,vartype = "ci",method="xlogit",df=degf(chs.dsgn))

## Data loading and crude cleaning

We select year 2014-2016 and variables relating to smoking and
insurance.

``` r
chs16 = read_sas("data/chs2016_public.sas7bdat")

chs16_filter = chs16 %>% 
  select(agegroup,generalhealth,insuredgateway16,insure16,insured,insure5,sickadvice16,sickplace,reasoner16,didntgetcare16,insaccept,didntfillin,medcost,everyday,numberperdaya,cost20cigarettes,smokeecig12m,smokeecig30days,ecighelpquit,didntfillin,timeunins16) %>% 
  mutate(year = 2016)


chs15 = read_sas("data/chs2015_public.sas7bdat")

chs15_filter = chs15 %>% 
  select(agegroup,generalhealth,insuredgateway15,insure15,insured,insure5,sickadvice15,sickplace,didntgetcare15,insaccept,didntfillin,medcost,everyday,numberperdaya,cost20cigarettes,didntfillin) %>% 
  mutate(timeunins15 = NA,
         reasoner15 = NA,
         smokeecig12m = NA,
         smokeecig30days = NA,
         ecighelpquit = NA,
         year = 2015) 


chs14 = read_sas("data/chs2014_public.sas7bdat")

chs14_filter = chs14 %>% 
  select(agegroup,generalhealth,insuredgateway14,insure14,insured,insure5,sickadvice14,didntgetcare14,everyday,numberperdaya,cost20cigarettes,smokeecig12m,smokeecig30days) %>% 
  mutate(timeunins14 = NA,
         reasoner14 = NA,
         sickplace4 = NA,
         insaccept = NA,
         didntfillin = NA,
         medcost = NA,
         ecighelpquit = NA,
         year = 2014) 


chs_14_16 = bind_rows(chs14_filter,chs15_filter,chs16_filter)
```