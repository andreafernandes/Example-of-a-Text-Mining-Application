# appetite-textmining-code


**Using [GATE](https://gate.ac.uk/family/) to text mine a data variable, which happens to be 'appetite'**

This repo is a record of my [JAPE](https://gate.ac.uk/sale/tao/splitch8.html#chap:jape) code to text mine the variable "appetite" within a bespoke psychiatric health record database, [CRIS](http://www.slam.nhs.uk/about/core-facilities/cris). 

It is mainly for my own benefit but do feel free to contact me for suggestions, questions or constructive feedback!


##About Author


###Details and Contact Information
| Author Name | Andrea Fernandes| 
|:--------|:--------------|
|Occupation| PhD student, 2nd Year|
|Email | andrea.fernandes@kcl.ac.uk|
|University|King's College London|
|Funding| MHRUK - Mental Health Research UK|

**********


Purpose of 'Appetite Variable' -   
This code will highlight the term 'appetite' AND any terms which collectively describe either 'good' or 'poor' appetite.


| Data source         | [CRIS](http://www.slam.nhs.uk/about/							core-facilities/cris) - Pseudonymised 							electronic psychiatric healthcare 							records| 
|:--------------------|:------------|
|Data extracted       | Clinical notes with the term 							'appetite' menioned were extracted 							 and processed to be read by GATE. 							 Contact author for details on this. |
|Text-mining software | [GATE](www.gate.ac.uk)|


*************
##Training Data

The following JAPE code and [gazateers](https://gate.ac.uk/sale/tao/splitch13.html) were generated after using 500 documents, each of which had the term 'appetite' mentioned and each of which was coded by two separate annotators to describe "good" or "bad appetite". 

Rules to code appetite were generated beforehand: [GOLD STANDARD RULES](https://cloud.githubusercontent.com/assets/10629155/12238166/95b70598-b87a-11e5-9fe6-dcefe1bd904b.png). 



#### GAZATEERS

A Gazateer is - basically -  a list of terms stored in a .lst file. 

I used 5 gazateers to help me pick up "good" or "bad appetite" (The list of terms were derived from the Training Set)



Gazateer 1: appetite_gazateer.lst file 

This gazateer that picks up "appetite" in all potential forms

```
Appetite
appetite
appettite
Appettite
appetitte
Appetitte
apetite
Apetite
apetitte
Apetitte
apettite
Apettite
appettitte
Appettitte
appetite:
Appetite:
appetite-
Appetite-
appetite

```
Gazateer 2: appetite poor describe.lst 

This is a list of terms that describe poor appetite. 

```
Absent 
absent
Decreasing
decreasing
Decreased
decreased
Decrease in
decrease in
deficit
Deficit
down
Down
gone
gone down
Gone
Gone down
loss of
Loss of
losing
Losing
loosing
lack of
Lack of
Loss
loss
Lost
lost
Low 
low
little
Little
Lacks
lacks
Lack
lack
Lacking 
lacking
No 
no
No interest
no interest
not great
not as good
not very well
Poor
poor
reduction
Reduction
Reduced
reduced
Smaller
smaller
Suppressed
suppressed
Suppress
suppress
Suppression
suppression
Worse
worse
Worsening
worsening
```

Gazateer 3: appetite good describe.lst

```
alright
Alright
denies problems with
denies issues with
eats well
Eats well
Eating well
eating well
excellent
Excellent
fine
Fine
fair
Fair
Good
good
great
Great
has appetite
Has appetite
healthy
Healthy
intact
Intact
Not too bad
not too bad
no problem
no problems
No problems
No problem
no concerns
No concerns
no concern
No concern
No difficulties
no difficulties
No difficulty
no difficulty
No issue
No issues
no issue
no issues
normal
Normal
ok
okay
OK
Ok
reasonable
Reasonable 
Regular
regular
stable
Stable
satisfactory
Satisfactory
steady
Steady
Unremarkable
unremarkable
Unimpaired
unimpaired
```

Gazateer 4: negation.lst

This .lst file was generated to be able to interpret negation terms used with "good appetite" or "bad appetite". 

For example, "he _DID NOT_ have bad appetite". I need to write JAPE to code this sentence as "NOT bad appeite"

```
never
Never
Denied
denied
did not
no
No
denies
Denies
does not report
Does not report
not been
not
Not
Not been
never had
Never had
Not having 
not having
was not
```

Gazateer 5: symptom_gazateer.lst 

These were symptoms I wanted to _IGNORE_ but which kept being mentioned with "appetite". I generate this file to then write JAPE code to ignore the below symptoms. 

```
sleep
Sleep
Memory
memory
Concentration
concentration
Weight
weight
Depression
depression
depressed
Depressed
Energy
energy
Mood
mood
```

These .lst files are then saved into a single  .def file, which looks like so:


```
appetite_gazateer.lst:appetite_mention
appetite_poor_describe.lst:appetite_description:poor
appetite_good_describe.lst:appetite_description:good
negation.lst:negationterms:negation
symptom_gazateer.lst:Symptom:symptoms
```
I saved this file as appetite.def

Notice that each of my .lst files are listed in this appetite.def file. 

Each listing is FOLLOWED BY a format similar to this:

xxxx.lst file: yyyyyy: zzzzzz 

The "yyyyyy" and "zzzzzz" are known as MAJOR TYPE and MINOR TYPE. 

In the JAPE code, i go one to use these MAJOR TYPE and MINOR TYPE terms to give my coding more flexibility. You'll see this below. 





#### JAPE CODE

OK so now that I have my list of terms that I am confident described "good" and "bad appetite" in CRIS, I will start on JAPE code. 

Jape code writing - Several days pass




######FINAL JAPE code


```
Phase: Appetite
Input: Lookup Token Split
Options: control = appelt



Rule: AppetiteThenDescriber
Priority: 10

(

	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly

	({Lookup.minorType == "good"} | 
 	 {Lookup.minorType == "poor"}):descriptorsOfAppetite

):AppetiteDescription
-->
:AppetiteDescription.Appetite = {rule = "AppetiteDescription1", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "no"}




Rule: AppetiteThenDescriber
Priority: 9
(

	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly

	 {Token.category == ":", Token.category != ","} 
	({Lookup.minorType == "good"} | 
  	 {Lookup.minorType == "poor"}):descriptorsOfAppetite

):AppetiteDescription
-->
:AppetiteDescription.Appetite = {rule = "AppetiteDescription2", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "no"}




Rule: DescriberThenAppetite
Priority: 9
(

	({Lookup.minorType == "good"} | 
	 {Lookup.minorType == "poor"}):descriptorsOfAppetite
	 {Token.category != ","}
	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly

):AppetiteDescription
-->
:AppetiteDescription.Appetite = {rule = "AppetiteDescription3", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "no"}




Rule: DescriberThenAppetite
Priority: 8
(

	({Lookup.minorType == "good"} | 
	 {Lookup.minorType == "poor"}):descriptorsOfAppetite
	 {Token.category != ","}
	({Token,!Split})[0,3]
	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly

):AppetiteDescription
-->
:AppetiteDescription.Appetite = {rule = "AppetiteDescription4", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "no"}




Rule: AppetiteThenDescriber
Priority: 7
(

	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly
	({Token,!Split})[0,3]
	 {Token.category != ","}
	({Lookup.minorType == "good"} | 
	 {Lookup.minorType == "poor"}):descriptorsOfAppetite

):AppetiteDescription
-->
:AppetiteDescription.Appetite = {rule = "AppetiteDescription5", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "no"}




Rule: AppetiteThenDescriber
Priority: 7
(

	({Lookup.minorType == negation}):Negationwords
	({Token,!Split})[0,3]
	({Lookup.minorType == "good"}  | 
	 {Lookup.minorType == "poor"}):descriptorsOfAppetite
	({Token,!Split})[0,3]
	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly

):AppetiteDescriptionwithNegation
-->
:AppetiteDescriptionwithNegation.Appetite = {rule = "AppetiteDescriptionwithNegation6", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "yes"}




Rule: AppetiteThenDescriber
Priority: 6
(

	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly	
	({Token,!Split})[0,3]
	({Lookup.minorType == negation}):Negationwords
	({Token,!Split})[0,3]
	({Lookup.minorType == "good"}  | 
	 {Lookup.minorType == "poor"}):descriptorsOfAppetite

):AppetiteDescriptionwithNegation
-->
:AppetiteDescriptionwithNegation.Appetite = {rule = "AppetiteDescriptionwithNegation7", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "yes"}




Rule: AppetiteThenDescriber
Priority: 6
(

	({Token.string =~ "[Nn]o"})
	({!Token.string == "change"})
	({Token,!Split})[0,3]
	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly
	({Token.string =~ "[Pp]roblems"})

):AppetiteDescriptionwithNegation
-->
:AppetiteDescriptionwithNegation.Appetite = {rule = "AppetiteDescriptionwithNegation8", AppetiteType = "good" ApplyNegation = "no"}
```
 


***********************************
# TESTING THE GAZATEER and JAPE CODE

We need to some Gold Standard data to test the above Gazateers and Code. 

### Gold Standard data 

500 documents extracted from CRIS. Each document contains the word "appetite"

These documents were annotated by two individuals to code good or bad appeite accoding to the [GOLD STANDARD RULES](https://cloud.githubusercontent.com/assets/10629155/12238166/95b70598-b87a-11e5-9fe6-dcefe1bd904b.png). 

Ensure that inter-rater reliability is above 85% at least to minimise erroneous performance testing. 


###Performance Testing

Testing on Gold Standard data

######Precision: 

######Recall:


Comparison to Machine Learning Process using GATE machine learning:


######Precision: 

######Recall:






