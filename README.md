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

Rules to code appetite were generated beforehand and were kept as simple as possible: ![GOLD STANDARD RULES](https://cloud.githubusercontent.com/assets/10629155/12238166/95b70598-b87a-11e5-9fe6-dcefe1bd904b.png) 



#### GAZATEERS

A Gazateer is - basically -  a list of terms stored in a .lst file. 

I used 5 gazateers to help me pick up "good" or "bad appetite" (The list of terms were derived from the Training Set). These are in the "gazateer" repo. 

Here's the list of .lst files I generated: 



###### Gazateer 1: appetite_gazateer.lst file 

###### Gazateer 2: appetite poor describe.lst 

###### Gazateer 3: appetite good describe.lst

###### Gazateer 4: negation.lst

###### Gazateer 5: symptom_gazateer.lst 


These .lst files are then saved into a single  .def file, which looks like so:


```
appetite_gazateer.lst:appetite_mention
appetite_poor_describe.lst:appetite_description:poor
appetite_good_describe.lst:appetite_description:good
negation.lst:negationterms:negation
symptom_gazateer.lst:Symptom:symptoms
```



#### JAPE CODE

This was created after going through the Training set and generating a set of "patterns" or themes in the text that appeared consistently or often in the documents, when describing appetite. 

Based on the themes I wrote the following JAPE code.



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

The inter-rater reliability was quite high - above 85% (but i don't have exact figures). 


###Final Performance

Testing on Gold Standard data (500 documents) generated a precision of 80% and a recall of 73%.








