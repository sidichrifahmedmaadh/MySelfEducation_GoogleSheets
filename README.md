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

Step 1: Open Apps Script

- In Google Sheets, go to Extensions > Apps Script.
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
            SpreadsheetApp.getUi().alert("Veuillez entrer une durée valide au format hh:mm (par exemple, 2:30).");
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
          SpreadsheetApp.getUi().alert("Veuillez entrer une durée au format hh:mm (par exemple, 2:30).");
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

Step 1: Initial Setup

- Enter the subject names in column A.
- Fill in the initial hours and minutes in columns B and C.

Step 2: Add New Hours

- In column D, enter a duration in the hh:mm format for a subject.
- The hours and minutes will automatically update in columns B and C.

Step 3: View Total

- Check the last cell in column D to see the cumulative total in hh:mm format.

Expected Results

	1.	After entering a duration in column D, columns B and C are updated:
(Insert a screenshot here showing the updated table.)
	2.	The last cell in column D displays the total in hh:mm format:
(Insert a screenshot here showing the total.)

Troubleshooting

- Formatting Error: Ensure that durations in column D are entered in the hh:mm format.
- Script Not Working: Verify that triggers are correctly set up and the sheet is named “Sheet1”.

Future Improvements

- Add a chart to visualize time spent on each subject.
- Automate reminders to meet study goals.

