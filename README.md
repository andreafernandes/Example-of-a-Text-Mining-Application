# appetite-textmining-code
Work in Progress


**Using GATE to text mine 'appetite'**

This repo is a record of my JAPE code to text mine variables within a bespoke psychiatric health record database. 


##About Author


###Details and Contact Information
| Author Name | Andrea Fernandes| 
|:--------|:--------------|
|Occupation| PhD student|
|Email | andrea.fernandes@kcl.ac.uk|
|University|King's College London|
|Funding| MHRUK - Mental Health Research UK|

**********


Purpose of 'Appetite Variable' -   
This code will highlight the term 'appetite' AND any terms which collectively describe either 'good' or 'poor' appetite.


| Data source         | CRIS - Pseudonymised electronic 							 psychiatric healthcare records| 
|:--------------------|:------------|
|Data extracted       | Clinical notes with the term 							'appetite' menioned were extracted 							 and processed to be read by GATE|
|Text-mining software | GATE (www.gate.ac.uk)|


*************


The following code and gazateers was generated after using 500 documents to generate patterns (from which the below rules were written) that would describe the user's appetite. 

GAZATEER LISTs 
 

![WordScreenshot] (/Users/andreafernandes/Google Drive/2_Research/PhD work/PhD work/Phase_2_Data_Extraction/Appetite_GATE_work/appetite/appetite version 3/Images_for_Git/Test.png)


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



**Breaking down the code**



```
Rule: AppetiteThenDescriber
Priority: 10

(

	({Token.string =~ "[Aa]ppetite", !Lookup.MajorType == "Symptom"}):SymptomAppetiteOnly

	({Lookup.minorType == "good"} | 
 	 {Lookup.minorType == "poor"}):descriptorsOfAppetite

):AppetiteDescription
-->
:AppetiteDescription.Appetite = {rule = "AppetiteDescription1", AppetiteType = :descriptorsOfAppetite.Lookup.minorType, ApplyNegation = "no"}
```

