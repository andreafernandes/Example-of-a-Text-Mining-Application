# Text Mining for Non-Professionals. 

## Basic Example: Using Text Mining to Code appetite (variable) in a dataset as "good" or "bad". 



**Using Text mining software - [GATE](https://gate.ac.uk/family/) - to extract a data variable (appetite) from free text clinical notes (and code appetite as good or bad).**


This is a record of my [JAPE](https://gate.ac.uk/sale/tao/splitch8.html#chap:jape) code to text mine the variable "appetite" within a bespoke psychiatric health record database, [CRIS](http://www.slam.nhs.uk/about/core-facilities/cris). 

It is mainly for my own records but also functions as a link to send out to anyone interested. 


####About Author
| Author Name | Andrea Fernandes| 
|:--------|:--------------:|
|Occupation| PhD student, 2nd Year|
|Email | andrea.fernandes@kcl.ac.uk|
|University|King's College London|
|Funding| MHRUK - Mental Health Research UK|



| **Data source**     | **[CRIS](http://www.slam.nhs.uk/									about/core-facilities/cris) - 									Pseudonymised electronic 									psychiatric healthcare records** | 
|:--------------------|:------------|
|**Data extracted**   | **Clinical notes with the term 							'appetite' menioned were extracted 							 and processed to be read by GATE. 							 Contact author for details on this.** |
|**Text-mining software** | **[GATE](www.gate.ac.uk)**|
|
|**Level of Training**| **I'd say Beginner Level**
							|**GATE 5 day training course: Included: how to create gazateers, how to write JAPE code and run Machine Learning algorithms on GATE**|



**********


**Purpose of 'Appetite Variable'**   
This code will highlight the term 'appetite' AND any terms which collectively describe either 'good' or 'poor' appetite.


*************
##Standard Steps to Build a Text Mining Application
I followed the following steps to create a functioning code to textmine appetite from CRIS records. 

I came up with these steps from a trial and error phase - trying to learn GATE (writing JAPE code), defining what I wanted the eventual produc to look like and understanding how data is recording in the data source. 

So these steps may be bespoke to my learning experience but I would apply them if I ever wanted to text-mine again. 


**1) Define the Problem/Variable**

Define and think through what it is you want to extract. Think of your variable outcome. What type of data would you like your outcome variable to display? What do you want this variable to inform you of? 

So try to write a set of rules, which  

 - Define your variable of interest and 
 - Inform of what you want out of your textmining code eventually


Make sure to give some examples, if you think this will help you define you problem/variable. 


**2) Scoping Exercise**

To help with understanding what it is what you want from the free-text data (or from your dataset) and to generate tools that you'll need to build your text mining application, conduct a scoping exercise. 

Go through around 500 to 1000 documents in your dataset which contain the data that you are interested in. 

During this exercise you aim is to meet each of the following:  

- Familiarise yourself with how your variable of interest is being mentioned in your dataset 
- do some exemplar coding of data; at least 500 documents
- make a note of and develop resolutions to ant issues you think you may face
- what are common terms/words associated with the main variable.
- Make a List of terms/words that you come across in a .lst files
- What terms make your variable of interest redundant or negate the variable
- Make a list of these terms that you like your application to exclude


**3) Re-define Problem**

Having done the scoping exercise, go back to your problem/variable definition. Read through it and see if you can add any extra examples, rules or notes to make your definition explicit. Give examples and lists of terms if you think it will help with the definition. 


**4) Thorough Consultation Process**

Set up a thorough consultation process with relevant colleagues and supervisors (and experts in the field) to make sure you've covered your bases with: explaing your variable, defining solution to overcome each issue you face to mining this variable. Show them the document you put together and make sure they understand what is on the document. 

Based on feedback, make any revisions to your document.



***At this point you should have:***


- ***A clear definition of your variable (Gold Standard Rules)***
- ***Lists: One or more lists (.lst files) of terms that describe each aspect of your variable (GATE calls these Gazateers).***
- ***JAPE code knowledge beginner level: Make sure you have had some GATE training to start up JAPE***

*If you are have the above then continue to the next steps:*

**5) Extract documents for Training Set and Gold Standard.** 

Extract 500 random documents for each set.

**6) Annotate Training Set**

Based on your polished rules from Step 3) annotate the Training Set set.

Ideally you should get two annotators to annotate your Training Set and check Inter-rater Reliability to minimise bias or skewed annotations. If performance from this is low, then you'll need repeat Steps 1 to 5, with a second annotator input. 


**7) Annotate the Gold Standard**


When you are happy with the training set, proceed to Annotating the Gold Standard the same way as you did with the Training set.  

**8) Write JAPE and Create Gazateers**

Based on your Gold Standard Rules, begin to write JAPE and make use of your lists. 

You can continuously test your JAPE and Gazateers out on smaller document sets to make sure the JAPE is doing what you want it to do. Just make sure these documents are not part of the training set or the Gold Standard. 

**9) Evaluate Application**

Test out final Gazateer list and JAPE on Gold-Standard



*************
##GATE Application to Extract Appetite


**Result of Scoping Exercise**

I developed a simple set of rules to code appetite: ![GOLD STANDARD RULES](https://cloud.githubusercontent.com/assets/10629155/12238166/95b70598-b87a-11e5-9fe6-dcefe1bd904b.png) 

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

This jape code was written after going through the Training set data (500 documents) and matched each Rule from the Gold Standard document.


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
## Testing the Application

We need to some Gold Standard data to test the above Gazateers and Code. 

The final application was run over the Gold Standard documents. 

###Final Performance

Testing on Gold Standard data (500 documents) generated a **_Precision_** of **80%** and a **_Recall_** of **73%.**



********

#Conclusions
GATE is a user friendly text mining application that can be used by students, given initial introductory training. In my application above, there is room for improvement. 

Further GATE machine learning applications, increased the precision to 83% and recall to 87%.










