## Study Hours Tracker - Google Sheets

This guide explains how to use a Google Sheets file to track your study hours and provides details about the Google Apps Script used to automate calculations.

Key Features

1. Track hours and minutes studied for different subjects.
2. Automatically add new hours and minutes into the “Hours” and “Minutes” columns.
3. Update the total hours and minutes at the bottom of the table.
4. Automatically convert minutes into additional hours when the total exceeds 60 minutes.

## Steps for Setup and Usage




## 1. Create the Table Structure

- Open Google Sheets.
- Create a table with the following columns:
- Subject / Science / Discipline: List of subjects or disciplines being studied.
- Hours: Total number of hours studied.
- Minutes: Total number of minutes studied.
- Add New Hours: Allows adding new durations in the hh:mm format.


Example Table: 
<br/><br/>
![Table](https://i.ibb.co/4gDC7SW/Capture-d-cran-2024-12-04-013538.png)


## 2. Create the Apps Script

<b> Step 1: Open Apps Script </b> 

- In Google Sheets, go to <b>Extensions</b> >  <b>Apps Script</b>.
- Delete any existing content and paste the following code:

```python
function updateTopSubjects(sheet) {
  const lastRow = sheet.getLastRow();
  const data = sheet.getRange(2, 1, lastRow - 2, 3).getValues(); // Exclut la dernière ligne
  const subjects = [];

  // Calcul des minutes totales pour chaque matière
  data.forEach(row => {
    const subject = row[0];
    const hours = parseInt(row[1], 10) || 0;
    const minutes = parseInt(row[2], 10) || 0;
    const totalMinutes = hours * 60 + minutes;

    if (subject) {
      subjects.push({ subject, totalMinutes });
    }
  });

  // Tri et affichage des 5 matières les plus étudiées
  subjects.sort((a, b) => b.totalMinutes - a.totalMinutes);
  const topSubjects = subjects.slice(0, 5).map(item => [item.subject, formatTime(item.totalMinutes)]);
  
  // Affiche les résultats à partir de la cellule G3
  sheet.getRange(3, 7, 5, 2).clearContent().setValues(topSubjects);
}

function formatTime(totalMinutes) {
  const hours = Math.floor(totalMinutes / 60);
  const minutes = totalMinutes % 60;
  return `${hours}:${minutes.toString().padStart(2, "0")}`;
}

function onEdit(e) {
  try {
    const sheet = e.source.getActiveSheet();
    const range = e.range;

    if (sheet.getName() === "Feuille 1" && range.getColumn() === 4 && range.getRow() > 1) {
      const newEntry = range.getValue();
      
      if (newEntry !== "") {
        const hoursCell = sheet.getRange(range.getRow(), 2); 
        const minutesCell = sheet.getRange(range.getRow(), 3);
        const timeParts = newEntry.toString().split(":");
        
        if (timeParts.length === 2) {
          const newHours = parseInt(timeParts[0], 10);
          const newMinutes = parseInt(timeParts[1], 10);

          if (isNaN(newHours) || isNaN(newMinutes)) {
            SpreadsheetApp.getUi().alert("Please enter a duration in hh:mm format (e.g. 2:30)).");
            range.setValue("");
            return;
          }

          let currentHours = parseInt(hoursCell.getValue(), 10) || 0;
          let currentMinutes = parseInt(minutesCell.getValue(), 10) || 0;

          currentHours += newHours;
          currentMinutes += newMinutes;

          if (currentMinutes >= 60) {
            const extraHours = Math.floor(currentMinutes / 60);
            currentMinutes %= 60;
            currentHours += extraHours;
          }

          hoursCell.setValue(currentHours);
          minutesCell.setValue(currentMinutes);
          range.setValue("");
          
          updateTopSubjects(sheet);
          updateTotalInLastCell(); // Appel de la fonction updateTotal ici
        } else {
          SpreadsheetApp.getUi().alert("Please enter a duration in hh:mm format (e.g. 2:30).");
          range.setValue("");
        }
      }
    }
  } catch (error) {
    SpreadsheetApp.getUi().alert("Une erreur est survenue : " + error.message);
  }
}

function updateTotalInLastCell() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const lastRow = sheet.getLastRow();

  if (lastRow < 3) return;

  const hoursRange = sheet.getRange(2, 2, lastRow - 2);
  const minutesRange = sheet.getRange(2, 3, lastRow - 2);

  const hoursData = hoursRange.getValues().flat();
  const minutesData = minutesRange.getValues().flat();

  let totalHours = 0;
  let totalMinutes = 0;

  hoursData.forEach(hour => {
    totalHours += parseInt(hour, 10) || 0;
  });

  minutesData.forEach(minute => {
    totalMinutes += parseInt(minute, 10) || 0;
  });

  const additionalHours = Math.floor(totalMinutes / 60);
  totalMinutes = totalMinutes % 60;
  totalHours += additionalHours;

  const totalCell = sheet.getRange(lastRow, 4);
  totalCell.setValue(`${totalHours}:${totalMinutes.toString().padStart(2, "0")}`);
}

```


## 3. Configure Triggers

1. In the Apps Script editor, go to Triggers (left-hand menu).
2. Set up a trigger to execute the onEdit function for every edit.

Usage

<b> Step 1: Initial Setup </b>  

- Enter the subject names in column A.
- Fill in the initial hours and minutes in columns B and C.

<b> Step 2: Add New Hours </b> 

- In column D, enter a duration in the hh:mm format for a subject.
- The hours and minutes will automatically update in columns B and C.

<b> Step 3: View Total </b> 

- Check the last cell in column D to see the cumulative total in hh:mm format.

Expected Results :

1. After entering a duration in column D, columns B and C are updated:
![Table](https://i.ibb.co/0BwX7Qb/Capture-d-cran-2024-12-04-023907.png)

3. The last cell in column D displays the total in hh:mm format:
![Table](https://i.ibb.co/B61kLKD/Capture-d-cran-2024-12-04-024203.png)

Troubleshooting

- Formatting Error: Ensure that durations in column D are entered in the hh:mm format.
- Script Not Working: Verify that triggers are correctly set up and the sheet is named “Sheet1”.



## 4. Future Improvements

- <b> Functionality: the Top 5 Most-Studied Subjects </b> 
<br/><br/>

The script automatically identifies and displays the top 5 subjects based on the total time studied (hours and minutes). These subjects are ranked in descending order of total study time, allowing you to track where you’ve focused the most effort.

Steps to Display the Top 5 Subjects

1. Tracking Study Time:
	- For each subject listed in the “Subject / Science / Discipline” column, the total study time (Hours and Minutes) is calculated.
	- These totals are converted to minutes for easy comparison.
2. Sorting the Data:
   	- The script compares the total time studied for all subjects and ranks them in descending order.
3. Displaying the Results:
   	- The top 5 most-studied subjects are displayed in a dedicated section (e.g., starting from cell G3).

Example
<br/><br/>
![Table](https://i.ibb.co/gvH6GGn/Capture-d-cran-2024-12-04-023019.png)


Using the Top 5 Subjects Section

1. Automatic Ranking:
	- Whenever you add or update time for any subject, the script recalculates the top 5 based on total study time.
2. Visualizing Your Efforts:
   	- The section provides an instant overview of your most-focused subjects, helping you adjust your study schedule if needed.
3. Customizable Display:
   	- You can change the starting cell (e.g., G3) in the script or increase the number of subjects displayed.


- <b> Functionality: Add a chart to visualize time spent on each subjectl </b> 
<br/><br/>

Example
<br/><br/>
![Table](https://i.ibb.co/FVDVmLM/Capture-d-cran-2024-12-04-023052.png)


<br/><br/><br/>
This Is [MY SELF EDUCATION](https://docs.google.com/spreadsheets/d/1ZML9h4zXsLC8KIx6g4DR-lsSJ7VzN-aj-Meo9Lrdt7w/edit?usp=sharing).

<p align="center">
   	<b>	
		If you like it, give the <a href="https://github.com/sidichrifahmedmaadh/MySelfEducation_GoogleSheets"> project </a>  a star on Github and <br/>
		share with your friends!! I will be happy with it! ❤️ <br/>
		I hope you learn something :).
	</b>
</p>
