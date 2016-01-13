# appetite-textmining-code


**Using Text mining software - [GATE](https://gate.ac.uk/family/) - to extract a data variable from free text clinical notes.**


This repo is a record of my [JAPE](https://gate.ac.uk/sale/tao/splitch8.html#chap:jape) code to text mine the variable "appetite" within a bespoke psychiatric health record database, [CRIS](http://www.slam.nhs.uk/about/core-facilities/cris). 

It is mainly for my own records and this archive functions as a link to send out to anyone interested. 

Do feel free to contact me for suggestions, questions or constructive feedback!


##About Author


###Details and Contact Information
| Author Name | Andrea Fernandes| 
|:--------|:--------------|
|Occupation| PhD student, 2nd Year|
|Email | andrea.fernandes@kcl.ac.uk|
|University|King's College London|
|Funding| MHRUK - Mental Health Research UK|

**********


**Purpose of 'Appetite Variable'**   
This code will highlight the term 'appetite' AND any terms which collectively describe either 'good' or 'poor' appetite.


| Data source         | [CRIS](http://www.slam.nhs.uk/about/							core-facilities/cris) - Pseudonymised 							electronic psychiatric healthcare 							records| 
|:--------------------|:------------|
|**Data extracted**       | **Clinical notes with the term 							'appetite' menioned were extracted 							 and processed to be read by GATE. 							 Contact author for details on this.** |
|**Text-mining software** | **[GATE](www.gate.ac.uk)**|


*************
I followed the **following steps** to create a functioning code to textmine appetite from CRIS records. 

**1) Define Problem**

Define and think through what it is you want to extract. Think of your variable outcome. What type of data would you like your outcome variable to display? What do you want this variable to inform you of? 

Write a set of rules that 

i) define your variable of interest and 
ii) what you want out of your textmining code eventually. Give some examples.


**2) Scoping Exercise**

To help with understanding what it is what you want from the free-text data, conduct a scoping exercise. Go through around 500 to 1000 documents, relevant to your variable of interest. 

During this exercise, familiarise yourself with how your variable of interest is being mentioned, what are common terms/words associated with this variable. What terms make your variable of interest redundant or negate the variable. 

**3) Re-define Problem**

After doing the scoping exercise, go back to your definition of and make any changes that you think will make your definition of the problem explicit. Give examples and lists of terms if you think it will help with the definition. 

**4) Thorough Consultation Process**

Set up a thorough consultation with relevant colleagues and supervisors (and experts in the field) at this point to make sure you've covered your bases with defining your variable of interest. Make any revisions at this point to your document. 

**At this stage you should happy with the definition of the Variable. You should have thought about and developed resolutions to all possible scenarios in which your variable could be mentioned in free-text.**

**5) Extract documents for Training Set and Gold Standard.** 

Extract 500 documents for each set.

**6) Annotate Training Set and Gold Standard**

Based on your document from 3) annotate the Training Set and Gold Standard sets on the same day. 

If the variable your interested is complicated, you should get two annotators to annotate your Training Set and check Inter-rater Reliability to minimise bias or skewed annotations. 

**7) Write JAPE and Create Gazateers**

Based on Training Set and Scoping Exercise Begin to Write JAPE and Create Necessary Gazateers. You can continuously test your JAPE and Gazateers out on smaller document sets to make sure the JAPE is doing what you want it to do. 

**8) Evaluate Application**

Test out final Gazateer list and JAPE on Gold-Standard



*************
###GATE Application to Extract Appetite


**Result of Scoping Exercise**

Rules to code appetite were generated were kept as simple as possible: ![GOLD STANDARD RULES](https://cloud.githubusercontent.com/assets/10629155/12238166/95b70598-b87a-11e5-9fe6-dcefe1bd904b.png) 

#### GAZATEERS

A Gazateer is - basically -  a list of terms stored in a .lst file. 

I used 5 gazateers to help me pick up "good" or "bad appetite" (The list of terms were derived from scoping exercises on CRIS, whose aim was to determine the types of words used to describe appetite in CRIS). 

Here's the list of .lst files I generated. These are in the "gazateer" repository: 



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

This jape code was written after going through the Training set data (500 documents) and generating a set of "patterns" or themes in the text that appeared consistently or often in the documents, when describing appetite. 

Based on the themes I wrote the following JAPE code, in chunks. I have each chunk priority to be run, according to how often these themes were mentioned in the text.



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








