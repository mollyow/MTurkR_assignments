MTurkR code for approving/awarding bonuses
================
Molly Offer-Westort
2018-01-08

Setup
-----

If starting a new session, load packages and set AWS access key.

``` r
Sys.setenv(AWS_ACCESS_KEY_ID = credentials_file$Access.Key.Id)
Sys.setenv(AWS_SECRET_ACCESS_KEY = credentials_file$Secret.Access.Key)
```

``` r
library(MTurkR)
library(foreign)
```

Approve assignments, award bonuses
----------------------------------

``` r
workers <- GetAssignments(hit = GetReviewableHITs()[,1]) # get all reviewable hits
workers <- workers[which(is.na(workers$ApprovalTime)),] # ignore already approved ones
ApproveAssignment(workers$AssignmentId) # Approve assignments
```

Match to survey (here, the filepath has already been saved as `f`).

``` r
dat <- read.csv(file=f, as.is=TRUE)
workers$surveycode <- trimws(workers$surveycode, which = "both")
```

Assign bonuses.

``` r
dat$bonus <- dat$stage1/100+ .1
dat$contrib_text <- paste0("Thank you for participating in this survey. Your bonus is calculated as follows: Your stage 1 earnings were ",
                           dat$stage1,
                           " tokens. Each token is worth $0.01. Your stage 1 bonus is $",
                           dat$stage1/100, 
                           ". In stage 2, you were randomly assigned the decider role. All deciders are assigned a fixed stage two earnings of 10 tokens. Each token is worth $0.01. Your total bonus is $",
                           dat$bonus, ".")


payout <- merge(workers, dat[, c("bonus", "contrib_text", "confirmation_code")], 
                by.x = "surveycode", 
                by.y = "confirmation_code", all.x=TRUE)

# Bonus workers with valid survey codes
payout <- payout[which(payout$surveycode!=""), ]


# Pay bonus
GrantBonus(workers=payout$WorkerId, assignments=payout$AssignmentId,
           amounts=payout$bonus, reasons=payout$contrib_text)
```
