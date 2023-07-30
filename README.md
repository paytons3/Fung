# Fung Institute MEng

# **Masters of Engineering Section Matching Project**
By John Jun and Payton Steiner


## **Section I: Project Introduction**

The goal of this project was to match 115 teams of 423 students total to sections for project classes 295 and 270K in Spring 2023 under the Masters of Engineering Program. It was proposed that we do this by coding a section matching algorithm where we could run in all the teams and end up with assigned classes for them. Numbers and capacities changed as the project went on, but these were the final numbers given before we completed the project: <br />

**295 Sections Notes** <br />
* 14 1-hour choices <br />
* Total number of sections that can seat 30+ students in Mudd 100 = 11 (sections: 3, 4, 5, 6, 7, 10, 11, 12, 13, 14, 15); @Felicia Rabang when you get a moment can you update the rooms on spreadsheet, i.e, no more Mudd 103, or give me edit access--thanks!:) <br />
* Total number of sections that can seat up to 30 students in Shires 311 = 3 (sections: 8, 9, 16) <br />
* Ideally there would be no more than 8 teams in one section <br />

**270K Sections Notes** <br />
* Currently 56 30-min choices with 8 sessions each <br />
* 2 teams share a slot as only attend 4x <br />
* Current total number of 112 sections (56x2) short 3 sections for 115 team sessions <br />
* Felicia working on scheduling 4 more to bring total to 116 <br />

As you can see from these notes, it was known that there were not enough sections for every team to be assigned so there was guaranteed manual section matching after the algorithm was run. We used Kaggle to work collaboratively on this project. This notebook contains the final code and results of the project.


## **Section II: Brainstorming & Research**

When first given the premise of the project, John found this paper on a similar student allocation problem. We used ideas and pseudocode from the paper to inspire our brainstorming process. Below is a screenshot from the paper of the general outline we started with: <br />

Some first notes we made on this pseudocode versus what we were doing were:
* We don't have lecturer preferences so will have to randomize choice of which teams get placement if there are ties. <br />
* If we are randomizing tie breaks, we won't have to remove students from sections. <br />
* Some sections may have a cap on number of teams rather than number of students <br />
* Google form allows groups to submit again if they want to change their preferences. before running alg, we need to delete duplicates by only keeping the response with the most recent timestamp. <br />


## **Section III: Coding the Algorithm**

### Section III.i – Data preparation

We first started by cleaning and editing a copy of the data. The data we used was in the form of team responses to a google form contained in this excel sheet. The process of preparing the data was as follows:
* Manually removed test responses we had in the excel sheet. 
* Imported the excel sheet as a Pandas dataframe.
* Set team id as the index of the dataframe.
* Split the dataframe into 2 separate dataframes: one with information in regards to the 295 sections and one in regards to the 270K sections.
* ***NOTE**: had to remember that the last two columns in the excel sheet were the N/A options for section preferences. <br />
We used RegEx to extract the section numbers and times for each section (located in the Section Matching section of the Kaggle notebook). 
* ***NOTE**: In future iterations of this project, it would be helpful for the Excel spreadsheets column names to have short and simple names, as this would make data cleaning easier. The Excel spreadsheet column names in our case were many sentences long, so doing RegEx to extract the title was difficult and time consuming.

### Section III.ii – Creating classes

We decided to go with an object oriented programming approach. Getting this code right was the bulk of the project. What we eventually ended up with was a Team Class and a Section Class that had two child classes – Section295 Class and Section270k Class – that inherited from the parent class. We used two child classes for Section because the 295 and 270K classes had different capacity requirements and that way we could set the class number, fail array, and section object variables as class variables for the two child classes. Below is a description of each class as well as the variables and functions contained within each class:

**Section Class**

Description: Class where each object is a course section

Variables: <br />
* self.name: _string_ <br />
  * Column name from the sections dataframe that corresponds to the appropriate section. <br />
* self.capacity: integer <br />
  * Number of teams that the section can contain. <br />
* self.numStudents: _integer_ <br />
  * Number of students currently assigned to the section. <br />
* self.team: _list of Team objects_ <br />
  * List of Team objects that have been assigned to the section. <br />
* self.worstPref: _integer_ <br />
  * The worst pref number (1-5 with 5 being the worst) of all teams currently assigned to the section. <br />
* self.worstPrefTeam: _Team object_ <br />
  * The team that has the worst pref number of all teams currently assigned to the section. <br />
  * Ties broken by being the most recent team added to the section. <br />
* self.currTime: _string_ <br />
  * Day of the week and time that the section is at. <br />

Functions: <br />
* __init__(self, sectionname, capacity, currTeam) <br />
  * Parameters: <br />
    * sectionname: _string_ <br />
    * capacity: _integer_ - Number of teams that Section can have <br />
    * currTime: _string_ - Day and time of the section <br />
  * Initializes Section object with sectionname as sectionname, capacity as capacity, numStudents as 0, teams as [] (empty list), worstPref as None, and currTime as currTime. <br />
* worstPrefCheck(self, team) <br />
  * Parameters: <br />
    * team: _Team object_ <br />
  * If worstPref hasn't been assigned yet, assign team as the team with the worst preference for the section. If it has been assigned but team has a worse preference than the current worst, reassign the worst preference team to team. Else, do nothing. <br />
* assignWorstPref(self, pref, team) <br />
  * Parameters: <br />
    * pref: _integer_ - Current preference number of the team <br />
    * team: _Team object_ <br />
  * Assign pref as this section's worst preference number and team as this section's team with worst preference. Worst refers to the preference number being the largest. <br />
* seekOtherWorst(self, sizeRequirement) <br />
  * Parameters: <br />
    * sizeRequirement: _integer_ <br />
  * Find a team that has the same or worse preference than the current team with a team size bigger than the current team (sizeRequirement). If found, make that team the new worst team and preference for this section. <br />
* replacingWorstPref(self, newTeam) <br />
  * Parameters: <br />
    * newTeam: _Team object_ <br />
  * Update the worstPref for a section after replacing a team. If the new team has a pref of 5, make that the new worstPref. Else, iterate through all teams to find the most recent team added that has the worstPref. <br />
* placeOldTeam(self, oldTeam) <br />
  * Parameters: <br />
    * oldTeam: _Team object_ <br />
  * If a team has been replaced, try to place the team in their next pref. If the team was already on their 5th pref, add them to the failed allocation list. <br />
* replace(self, oldTeam, newTeam) <br />
  * Parameters: <br />
    * oldTeam: _Team object_ <br />
    * newTeam: _Team object_ <br />
  * Replace a team (oldTeam) in the section that has a worse pref with newTeam, then run place on oldTeam and assign newTeam. <br />

**Section 295 Class**

Description: Subclass of Section where each object is a 295 course section

Variables: <br />
self.classNumber: _string_ <br />
Class section number. <br />
self.fail_array: _array of Team objects_ <br />
Array of team objects that were not assigned to any 295 class section.  <br />
self.section_objects: _array of Section objects_ <br />
Array of all Section295 objects.  <br />
cap: _integer_ <br />
Number of teams each section can have. <br />

Functions: <br />
__init__(self, sectionname, currTime) <br />
Parameters: <br />
Sectionname: _string_ <br />
currTime: _string_ - Day and time of the section <br />
Initializes object with classNumber as ‘295’, faily_array as fail_allocation_295, section_objects as section_295_objects, and cap as 9. <br />
super.__init__(sectionname, cap, currTime) <br />
Parameters <br />
sectionname: _string_ <br />
cap: _integer_ <br />
currTime: _string_ - Day and time of the section <br />
Sets object as child object of Section class with sectionname as sectionname, capacity as 9, and currTime as currTime. <br />
assign(self, team) <br />
Parameters: <br />
team: _Team object_ <br />
Assign team to this section and update instance variables accordingly. <br />
place(self, team) <br />
Parameters: <br />
team: _Team object_ <br />
Check if team can be placed in this section. If it can, assign it to this section and update variables accordingly. If on team’s 5th preference, add team to the fail allocation list. If preference is less than 5, move on to team’s next preference section. <br />

**Section 270k Class**

Description: Subclass of Section where each object is a 270k course section

Variables: <br />
self.classNumber: _string_ <br />
Class section number. <br />
self.fail_array: _array of Team objects_ <br />
Array of team objects that were not assigned to any 270K class section. <br />
self.section_objects: _array of Section objects_ <br />
Array of all Section270k objects <br />
self.teamObjectsFor295: <br />
List of assigned Section 295 objects so that they can be checked for time conflicts. <br />
cap: _integer_ <br />
Number of teams each section can have. <br />

Functions: <br />
__init__(self, sectionname, currTime) <br />
Parameters: <br />
sectionname: _string_ <br />
currTime: _string_ - Day and time of the section <br />
Initializes object with classNumber as ‘270k’, faily_array as fail_allocation_270k, section_objects as section_270k_objects, teamObjectsFor295 as teams_for_295, and cap as 9. <br />
super.__init__(sectionname, 2, currTime) <br />
Parameters: <br />
sectionname: _string_ <br />
cap: _integer_ <br />
currTime: _string_ - Day and time of the section <br />
Sets object as child object of Section class with sectionname as sectionname, capacity as 2, and currTime as currTime. <br />
assign(self, team) <br />
Parameters: <br />
team: _Team object_ <br />
Assign team to this section and update instance variables accordingly. <br />
timeConflictCheck(self, team) <br />
Parameters: <br />
team: _Team object_ <br />
Returns True if the current section has a time conflict with the given team's 295 assigned section. Returns False if there is no conflict. <br />
place(self, team) <br />
Parameters: <br />
team: _Team object_ <br />
Check if team can be placed in this section. If it can, assign it to this section and update variables accordingly. If on team’s 5th preference, add team to the fail allocation list. If preference is less than 5, move on to team’s next preference section. <br />

**Team Class**

Description: Class where each object is a team in the Masters of Engineering program

Variables: <br />
self.teamid: _string_ <br />
Teamid as listed in the teamid column of the sections dataframe. <br />
self.studentid: _string_ <br />
Studentid of the team member that filled out the Google form for their team as listed in the sections dataframe. <br />
self.size: _int_ <br />
Number of students on the team. <br />
self.teamindex: _string_ <br />
Index of the row in the sections dataframe that contains this team. <br />
self.currPref: _integer_ <br />
Current section preference of the team (1-5 starting with 1). <br />
self.currSection: _integer_ <br />
Current section that corresponds to the currPref number which is being checked to see if the team can be assigned to. <br />
self.preferences: _list of integers_ <br />
List of integers where the index corresponds to preference number and the integer itself corresponds to the column index of a section. <br />
self.currentTime: _string_ <br />
Current day of the week and time that the assigned section is at. <br />

Functions: <br />
__init__(self, teamid, studentid, size, teamindex) <br />
Parameters: <br />
teamid: _string_ <br />
studentid: _string_ <br />
size: _integer_ <br />
teamindex: _integer_ <br />
Initializes object with instance variables currPref as 1 and currSection, preferences, and currentTime as “None”. <br />
createPreferences(self, df) <br />
Parameters: <br />
df: _Pandas dataframe_ <br />
Create an ordered array of 5 numbers that are column indices for corresponding section preferences in df. <br />
addPref(self) <br />
Increases team.currPref by 1 to indicate that we have moved to the teams' next best preference choice. <br />
moveToNextSection(self, section_objects, classNumber, fail_array) <br />
Parameters: <br />
section_objects: _array of Section objects_ <br />
classNumber: _string_ <br />
fail_array: _array of Team objects_ <br />
Try placing team (self) in their next preferred section. <br />
failed(self, failed_during, failed_array) <br />
Parameters: <br />
failed_during: _string_ <br />
***NOTE**: can get rid of this parameter, was previously used in print statement that was removed <br />
failed_array: array of Team objects <br />
Adds team (self) to appropriate section fail array <br />


## **Section IV: Section Matching**

When creating section objects one of our first steps was randomizing the order of the rows in the sections dataframe. We did this to randomize which teams would get their preferences over others. We then created the following empty arrays: <br />
section_295_objects – array of Section295 objects <br />
section_270k_objects – array of Section270k objects <br />
fail_allocation_295 – array of teams not assigned to a 295 class section <br />
fail_allocation_270k – array of teams not assigned to a 270k class section <br />

We first created Section295 objects. We extracted the times of sections using RegEx and then created Section295 objects with the RegEx time as currTime and the column name as sectionname. We then created Team objects using each row from the sections dataframe. After creating a single Team object, we assigned the preferences array for the team using the createPreferences function of the Team class. We then added the team to the teams_for_295 array. Lastly, we indexed into section_295_objects using the team’s first preference and ran the place function of the Section295 class using the team object as the argument.

For assigning teams to 270K sections we repeated the same process as above with Section270k objects.


## **Section V: Testing & Results**

To create test cases, we generated fake teams with randomized preferences, and ran them through our algorithm. The results from these test cases were generally better than the actual final result, as the actual final result had a lot of popularity bias that caused certain sections to have too much interest, and some to have very few or none. 

We ran tests to confirm that there were not repeated teams in assignments and to make sure all teams were accounted for as either assigned or in the fail array. We also wrote code to verify no time conflicts between 295 and 270K sections for a team. We had to check for repeats because we originally struggled with a bug where some teams were appearing multiple times. Results of the tests were as follows: <br />
For the 295 sections, we ended up with 109 assigned teams and 6 unassigned teams. <br />
For the 270K sections, we ended up with 89 teams and 26 unassigned teams. <br />
There was 1 team that was not assigned to either class. <br />
It was verified that there were no time conflicts among the two sections for any time. <br />


## **Section VI: Manual Editing**

Despite aiming to create a fully automated process, we discovered that there needed to be some manual data cleaning prior to inputting the data into the algorithm.
This was because: <br />
Some students put less than 5 preferences. <br />
There were specific “exception” teams that requested accommodation in order to be able to work with everyone’s schedule. <br />
One team really had one possible section that worked for all of them. The way we combatted this was by always placing them first after random ordering, so they would always get their first choice. <br />
Many sections were incredibly popular, and many were very unpopular. What we ended up doing was if for example a team placed a popular section as their 1st choice and an unpopular section as their 4th choice, we manually swapped their choices. This way they would still get a top 5 preference, and give up the popular one. If we attempted to give everyone the popular choices, neither the popular or unpopular choices would have ended up being filled. <br />


## **Section VII: Improvements to be made**

With parts of this process being manual, and other parts being automated, there are disconnects in the process that could be optimized in future iterations. These include: <br />
Formatting the Excel spreadsheet to have shorter and easier to read columns <br />
Enforcing deciding on 5 preferences, except for when having extenuating circumstances. <br />
Trying to create more availability for popular time slots (such as adding additional lecture halls for the 11 am’s) <br />
On the Google form, list certain sections as historically popular, and therefore harder to get into. This may incentivize some teams to play it safe and choose other sections. <br />


## **Links**

Google Colab notebook <br />
Input data (results from MEng preference survey that teams filled out) <br />
Algorithm structure ideas and pseudocode <br />
Google form responses from teams <br />

