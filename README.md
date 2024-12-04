## Study Hours Tracker - Google Sheets

This guide explains how to use a Google Sheets file to track your study hours and provides details about the Google Apps Script used to automate calculations.

Key Features

	1.	Track hours and minutes studied for different subjects.
	2.	Automatically add new hours and minutes into the “Hours” and “Minutes” columns.
	3.	Update the total hours and minutes at the bottom of the table.
	4.	Automatically convert minutes into additional hours when the total exceeds 60 minutes.

Steps for Setup and Usage

1. Create the Table Structure

	1. Open Google Sheets.
	2. Create a table with the following columns:
	3. Subject / Science / Discipline: List of subjects or disciplines being studied.
	3. Hours: Total number of hours studied.
	4. Minutes: Total number of minutes studied.
	5. Add New Hours: Allows adding new durations in the hh:mm format.

Example Table: <br/>
![Table](https://i.ibb.co/4gDC7SW/Capture-d-cran-2024-12-04-013538.png)


2. Create the Apps Script

Step 1: Open Apps Script

	1.	In Google Sheets, go to Extensions > Apps Script.
	2.	Delete any existing content and paste the following code:

function onEdit(e) {
  try {
    const sheet = e.source.getActiveSheet();
    const range = e.range;

    // Check if the edit is in column D
    if (sheet.getName() === "Sheet1" && range.getColumn() === 4 && range.getRow() > 1) {
      const newEntry = range.getValue(); // Value entered in column D

      // Update the totals
      updateTotalInLastCell(sheet);

      if (newEntry !== "") {
        const hoursCell = sheet.getRange(range.getRow(), 2);   // Column B
        const minutesCell = sheet.getRange(range.getRow(), 3); // Column C

        const timeParts = newEntry.toString().split(":");
        if (timeParts.length === 2) {
          const newHours = parseInt(timeParts[0], 10);
          const newMinutes = parseInt(timeParts[1], 10);

          if (isNaN(newHours) || isNaN(newMinutes)) {
            SpreadsheetApp.getUi().alert("Enter a valid duration in the format hh:mm (e.g., 2:30).");
            range.setValue(""); // Clear the cell in case of error
            return;
          }

          let currentHours = parseInt(hoursCell.getValue(), 10) || 0;
          let currentMinutes = parseInt(minutesCell.getValue(), 10) || 0;

          // Add new hours and minutes
          currentHours += newHours;
          currentMinutes += newMinutes;

          if (currentMinutes >= 60) {
            const extraHours = Math.floor(currentMinutes / 60);
            currentMinutes = currentMinutes % 60;
            currentHours += extraHours;
          }

          hoursCell.setValue(currentHours);
          minutesCell.setValue(currentMinutes);

          range.setValue(""); // Clear the "Add New Hours" cell
        } else {
          SpreadsheetApp.getUi().alert("Please enter a valid duration in the format hh:mm.");
          range.setValue(""); // Clear the cell in case of incorrect input
        }
      }
    }
  } catch (error) {
    SpreadsheetApp.getUi().alert("An error occurred: " + error.message);
  }
}

function updateTotalInLastCell(sheet) {
  const lastRow = sheet.getLastRow();
  const hoursTotal = parseInt(sheet.getRange(lastRow, 2).getValue(), 10) || 0;
  const minutesTotal = parseInt(sheet.getRange(lastRow, 3).getValue(), 10) || 0;

  const additionalHours = Math.floor(minutesTotal / 60);
  const remainingMinutes = minutesTotal % 60;
  const totalHours = hoursTotal + additionalHours;

  const totalCell = sheet.getRange(lastRow, 4); // Last cell in column D
  totalCell.setValue(${totalHours}:${remainingMinutes.toString().padStart(2, "0")});
}

3. Configure Triggers

	1.	In the Apps Script editor, go to Triggers (left-hand menu).
	2.	Set up a trigger to execute the onEdit function for every edit.

Usage

Step 1: Initial Setup

	•	Enter the subject names in column A.
	•	Fill in the initial hours and minutes in columns B and C.

Step 2: Add New Hours

	•	In column D, enter a duration in the hh:mm format for a subject.
	•	The hours and minutes will automatically update in columns B and C.

Step 3: View Total

	•	Check the last cell in column D to see the cumulative total in hh:mm format.

Expected Results

	1.	After entering a duration in column D, columns B and C are updated:
(Insert a screenshot here showing the updated table.)
	2.	The last cell in column D displays the total in hh:mm format:
(Insert a screenshot here showing the total.)

Troubleshooting

	•	Formatting Error: Ensure that durations in column D are entered in the hh:mm format.
	•	Script Not Working: Verify that triggers are correctly set up and the sheet is named “Sheet1”.

Future Improvements

	•	Add a chart to visualize time spent on each subject.
	•	Automate reminders to meet study goals.

Feel free to customize this guide further or add your own screenshots to make it clearer. Let me know if you need additional help!
