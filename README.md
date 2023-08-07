# Fung Institute Masters of Engineering Section Matching Project
By John Jun and Payton Steiner

## Project Overview
Section I: Project Introduction <br />
Section II: Brainstorming & Research <br />
Section III: Coding the Algorithm <br />
* Section III.i – Data preparation
* Section III.ii – Creating classes <br />

Section IV: Section Matching <br />
Section V: Testing & Results <br />
Section VI: Manual Editing <br />
Section VII: Conclusions <br />

## References
[Google Colab notebook](https://colab.research.google.com/drive/1IjW0RdAJp-4tP8bG6YhmaB0aYn_9B_qo?authuser=1#scrollTo=azl9gfrCS_yb) <br />
[Algorithm structure ideas and pseudocode](https://www.cs.cmu.edu/~dabraham/papers/aim04.pdf) <br />
[Input data: Google form responses from teams](https://docs.google.com/spreadsheets/d/1mHUz5vffcjR6a_9w4dDLRMLor9nga1BHNyQRbX4lmns/edit?resourcekey=undefined#gid=1663346299) <br />

## **Section I: Project Introduction**

The goal of this project was to match 115 teams of 423 students total to sections for project classes 295 and 270K in Spring 2023 under the Masters of Engineering Program. It was proposed that we do this by coding a section matching algorithm where we could run in all the teams and end up with assigned classes for them. Numbers and capacities changed as the project went on, but these were the final numbers given before we completed the project:

**295 Sections Notes** 
* 14 1-hour choices 
* Total number of sections that can seat 30+ students in Mudd 100 = 11 (sections: 3, 4, 5, 6, 7, 10, 11, 12, 13, 14, 15); @Felicia Rabang when you get a moment can you update the rooms on spreadsheet, i.e, no more Mudd 103, or give me edit access--thanks!:)
* Total number of sections that can seat up to 30 students in Shires 311 = 3 (sections: 8, 9, 16)
* Ideally there would be no more than 8 teams in one section

**270K Sections Notes**
* Currently 56 30-min choices with 8 sessions each
* 2 teams share a slot as only attend 4x
* Current total number of 112 sections (56x2) short 3 sections for 115 team sessions
* Felicia working on scheduling 4 more to bring total to 116

As you can see from these notes, it was known that there were not enough sections for every team to be assigned so there was guaranteed manual section matching after the algorithm was run. We used Kaggle to work collaboratively on this project. This notebook contains the final code and results of the project.


## **Section II: Brainstorming & Research**

When first given the premise of the project, John found a paper on a similar student allocation problem. We used ideas and pseudocode from the paper to inspire our brainstorming process. ([This](https://www.cs.cmu.edu/~dabraham/papers/aim04.pdf) is the link to the paper referenced.)

Some first notes we made on this pseudocode versus what we were doing were:
* We don't have lecturer preferences so will have to randomize choice of which teams get placement if there are ties.
* If we are randomizing tie breaks, we won't have to remove students from sections.
* Some sections may have a cap on number of teams rather than number of students
* Google form allows groups to submit again if they want to change their preferences. before running alg, we need to delete duplicates by only keeping the response with the most recent timestamp.


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

Description: Class where each object is a course section.

Variables:
* self.name: _string_
  * Column name from the sections dataframe that corresponds to the appropriate section.
* self.capacity: integer
  * Number of teams that the section can contain.
* self.numStudents: _integer_
  * Number of students currently assigned to the section.
* self.team: _list of Team objects_ 
  * List of Team objects that have been assigned to the section.
* self.worstPref: _integer_ 
  * The worst pref number (1-5 with 5 being the worst) of all teams currently assigned to the section. 
* self.worstPrefTeam: _Team object_ 
  * The team that has the worst pref number of all teams currently assigned to the section. 
  * Ties broken by being the most recent team added to the section. 
* self.currTime: _string_ 
  * Day of the week and time that the section is at.

Functions:
* __init__(self, sectionname, capacity, currTeam)
  * Parameters:
    * sectionname: _string_ 
    * capacity: _integer_ - Number of teams that Section can have
    * currTime: _string_ - Day and time of the section
  * Initializes Section object with sectionname as sectionname, capacity as capacity, numStudents as 0, teams as [] (empty list), worstPref as None, and currTime as currTime.
* worstPrefCheck(self, team)
  * Parameters:
    * team: _Team object_ 
  * If worstPref hasn't been assigned yet, assign team as the team with the worst preference for the section. If it has been assigned but team has a worse preference than the current worst, reassign the worst preference team to team. Else, do nothing.
* assignWorstPref(self, pref, team)
  * Parameters:
    * pref: _integer_ - Current preference number of the team
    * team: _Team object_
  * Assign pref as this section's worst preference number and team as this section's team with worst preference. Worst refers to the preference number being the largest.
* seekOtherWorst(self, sizeRequirement)
  * Parameters:
    * sizeRequirement: _integer_
  * Find a team that has the same or worse preference than the current team with a team size bigger than the current team (sizeRequirement). If found, make that team the new worst team and preference for this section.
* replacingWorstPref(self, newTeam)
  * Parameters: 
    * newTeam: _Team object_ 
  * Update the worstPref for a section after replacing a team. If the new team has a pref of 5, make that the new worstPref. Else, iterate through all teams to find the most recent team added that has the worstPref.
* placeOldTeam(self, oldTeam)
  * Parameters:
    * oldTeam: _Team object_
  * If a team has been replaced, try to place the team in their next pref. If the team was already on their 5th pref, add them to the failed allocation list.
* replace(self, oldTeam, newTeam)
  * Parameters:
    * oldTeam: _Team object_ 
    * newTeam: _Team object_ 
  * Replace a team (oldTeam) in the section that has a worse pref with newTeam, then run place on oldTeam and assign newTeam.

**Section 295 Class**

Description: Subclass of Section where each object is a 295 course section

Variables:
* self.classNumber: _string_
  * Class section number.
* self.fail_array: _array of Team objects_ 
  * Array of team objects that were not assigned to any 295 class section.
* self.section_objects: _array of Section objects_
  * Array of all Section295 objects.
* cap: _integer_ 
  * Number of teams each section can have.

Functions: 
* __init__(self, sectionname, currTime)
  * Parameters: 
    * Sectionname: _string_ 
    * currTime: _string_ - Day and time of the section
  * Initializes object with classNumber as ‘295’, faily_array as fail_allocation_295, section_objects as section_295_objects, and cap as 9. 
* super.__init__(sectionname, cap, currTime) 
  * Parameters 
    * sectionname: _string_ 
    * cap: _integer_
    * currTime: _string_ - Day and time of the section <br />
  * Sets object as child object of Section class with sectionname as sectionname, capacity as 9, and currTime as currTime.
* assign(self, team)
  * Parameters: 
    * team: _Team object_
  * Assign team to this section and update instance variables accordingly.
* place(self, team) 
  * Parameters: 
    * team: _Team object_ 
  * Check if team can be placed in this section. If it can, assign it to this section and update variables accordingly. If on team’s 5th preference, add team to the fail allocation list. If preference is less than 5, move on to team’s next preference section. 

**Section 270k Class**

Description: Subclass of Section where each object is a 270k course section

Variables:
* self.classNumber: _string_ 
  * Class section number.
* self.fail_array: _array of Team objects_ 
  * Array of team objects that were not assigned to any 270K class section.
* self.section_objects: _array of Section objects_ 
  * Array of all Section270k objects 
* self.teamObjectsFor295:
  * List of assigned Section 295 objects so that they can be checked for time conflicts.
* cap: _integer_ 
  * Number of teams each section can have.

Functions: 
* __init__(self, sectionname, currTime)
  * Parameters:
    * sectionname: _string_ 
    * currTime: _string_ - Day and time of the section
  * Initializes object with classNumber as ‘270k’, faily_array as fail_allocation_270k, section_objects as section_270k_objects, teamObjectsFor295 as teams_for_295, and cap as 9.
* super.__init__(sectionname, 2, currTime)
  * Parameters:
    * sectionname: _string_ 
    * cap: _integer_ 
    * currTime: _string_ - Day and time of the section 
  * Sets object as child object of Section class with sectionname as sectionname, capacity as 2, and currTime as currTime.
* assign(self, team) 
  * Parameters:
    * team: _Team object_
  * Assign team to this section and update instance variables accordingly.
* timeConflictCheck(self, team)
  * Parameters: 
    * team: _Team object_ 
  * Returns True if the current section has a time conflict with the given team's 295 assigned section. Returns False if there is no conflict.
* place(self, team)
  * Parameters: 
    * team: _Team object_ 
  * Check if team can be placed in this section. If it can, assign it to this section and update variables accordingly. If on team’s 5th preference, add team to the fail allocation list. If preference is less than 5, move on to team’s next preference section.

**Team Class**

Description: Class where each object is a team in the Masters of Engineering program

Variables:
* self.teamid: _string_
  * Teamid as listed in the teamid column of the sections dataframe.
* self.studentid: _string_
  * Studentid of the team member that filled out the Google form for their team as listed in the sections dataframe. 
* self.size: _int_ 
  * Number of students on the team. 
* self.teamindex: _string_
  * Index of the row in the sections dataframe that contains this team.
* self.currPref: _integer_ 
  * Current section preference of the team (1-5 starting with 1).
* self.currSection: _integer_ 
  * Current section that corresponds to the currPref number which is being checked to see if the team can be assigned to.
* self.preferences: _list of integers_ 
  * List of integers where the index corresponds to preference number and the integer itself corresponds to the column index of a section. 
* self.currentTime: _string_ 
  * Current day of the week and time that the assigned section is at. 

Functions: 
* __init__(self, teamid, studentid, size, teamindex) 
  * Parameters: 
    * teamid: _string_
    * studentid: _string_ 
    * size: _integer_ 
    * teamindex: _integer_
  * Initializes object with instance variables currPref as 1 and currSection, preferences, and currentTime as “None”.
* createPreferences(self, df)
  * Parameters: 
    * df: _Pandas dataframe_ 
  * Create an ordered array of 5 numbers that are column indices for corresponding section preferences in df.
* addPref(self) 
  * Increases team.currPref by 1 to indicate that we have moved to the teams' next best preference choice. 
* moveToNextSection(self, section_objects, classNumber, fail_array) 
  * Parameters: 
    * section_objects: _array of Section objects_
    * classNumber: _string_
    * fail_array: _array of Team objects_ 
  * Try placing team (self) in their next preferred section.
* failed(self, failed_during, failed_array)
  * Parameters: 
    * failed_during: _string_
      * ***NOTE**: can get rid of this parameter, was previously used in print statement that was removed
    * failed_array: array of Team objects
  * Adds team (self) to appropriate section fail array.


## **Section IV: Section Matching**

When creating section objects one of our first steps was randomizing the order of the rows in the sections dataframe. We did this to randomize which teams would get their preferences over others. We then created the following empty arrays: 
* section_295_objects – Array of Section295 objects
* section_270k_objects – Array of Section270k objects 
* fail_allocation_295 – Array of teams not assigned to a 295 class section 
* fail_allocation_270k – Array of teams not assigned to a 270k class section 

We first created Section295 objects. We extracted the times of sections using RegEx and then created Section295 objects with the RegEx time as currTime and the column name as sectionname. We then created Team objects using each row from the sections dataframe. After creating a single Team object, we assigned the preferences array for the team using the createPreferences function of the Team class. We then added the team to the teams_for_295 array. Lastly, we indexed into section_295_objects using the team’s first preference and ran the place function of the Section295 class using the team object as the argument.

For assigning teams to 270K sections we repeated the same process as above with Section270k objects.


## **Section V: Testing & Results**

To create test cases, we generated fake teams with randomized preferences, and ran them through our algorithm. The results from these test cases were generally better than the actual final result, as the actual final result had a lot of popularity bias that caused certain sections to have too much interest, and some to have very few or none. 

We ran tests to confirm that there were not repeated teams in assignments and to make sure all teams were accounted for as either assigned or in the fail array. We also wrote code to verify no time conflicts between 295 and 270K sections for a team. We had to check for repeats because we originally struggled with a bug where some teams were appearing multiple times. Results of the tests were as follows:
* For the 295 sections, we ended up with 109 assigned teams and 6 unassigned teams. 
* For the 270K sections, we ended up with 89 teams and 26 unassigned teams.
* There was 1 team that was not assigned to either class. 
* It was verified that there were no time conflicts among the two sections for any time. 


## **Section VI: Manual Editing**

Despite aiming to create a fully automated process, we discovered that there needed to be some manual data cleaning prior to inputting the data into the algorithm. <br />
This was because:
* Some students put less than 5 preferences.
* There were specific “exception” teams that requested accommodation in order to be able to work with everyone’s schedule.
* One team really had one possible section that worked for all of them. The way we combatted this was by always placing them first after random ordering, so they would always get their first choice. 
* Many sections were incredibly popular, and many were very unpopular. What we ended up doing was if for example a team placed a popular section as their 1st choice and an unpopular section as their 4th choice, we manually swapped their choices. This way they would still get a top 5 preference, and give up the popular one. If we attempted to give everyone the popular choices, neither the popular or unpopular choices would have ended up being filled.


## **Section VII: Conclusions**
This project was successful in matching teams of students in the UC Berkeley MEng program for Spring 2023 project courses. We hope that our end product will continue to be adapted and used for MEng course scheduling moving forward and that this documentation serves as a useful guide. 

As this was the first attempt in automizing this course scheduling process, we learned a lot about the needs for this project. Perhaps the most important thing taken away was that this is a project where there will inevitably be unexpected changes that we must adapt to, such as the addition of new sections or the changing of teams. We did our best to create code that can handle changes for future iterations of this project specifically, such as a different max capacity for classes.  However, it should be noted that the code used in this project is very tailored towards this particular scheduling scenario. For those looking to use this project as guidance for their own course scheduling needs we hope that our general process will be helpful, but the code will need to be adapted for your speficic needs.

With parts of this process being manual and other parts being automated, there are disconnects in the process that could be optimized in future iterations. For those taking on our specific project in the future, here are some notes of improvement that we have noted:
* Formatting the Excel spreadsheet to have shorter and easier to read columns.
* Enforcing deciding on 5 preferences, except for when having extenuating circumstances. 
* Trying to create more availability for popular time slots (such as adding additional lecture halls for the 11 am’s).
* On the Google form, list certain sections as historically popular, and therefore harder to get into. This may incentivize some teams to play it safe and choose other sections.

We thank you for viewing our guide and wish you luck in your course scheduling endeavors!
