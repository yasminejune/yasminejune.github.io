# Class Alert Pineapple
#### Video Demo:  <https://youtu.be/2Nedqu-HHuo>

## Purpose
The purpose of this project is for users to have an easy alert system when a [Pineapple](https://www.pineapple.uk.com/pages/studio-classes-timetable) dance class becomes available.

The app allows users to specify the teacher and the class date using a webapp, for which they wish to set up an alert for. Once the alert has been set, an API scheduler checks the availability of the class every 10 minutes. If the class status changes, the script will trigger a telegram blast sent out to every user who is connected to the telegram bot. Once the user is notified of a class becoming newly available, he can then book his slot manually.

The current set-up only allows users to define one alert at a time. The reason for this design-choice is simplicity. Although multiple alerts could be scheduled using the API, the website would have had to be intricate, and the backend able to distinguish between different alerts. Since my use has historically always been limited to one alert at a time, this app runs on one alert only.

Although only one alert can be set up at a time, multiple users can receive the alert. The telegram prompt

## Structure
The project runs from a single python script. This script initiates both the webapp, the scraper, and launches the telegram prompt, when needed.

The webapp provides the user interface to select the class, for which the class alert has been set up. The webapp has two different websites: one for setting the alert, and one for stopping all alerts.

The project is run on heroku, which is why users must include a Procfile in their directory to launch the heroku app.

Finally, any change in class status uses a telegram bot to blast a message to all connected users.

> [!IMPORTANT]
> Users must set up their own telegram bot, and insert the chatbot ID and key in the application.py code.

## Explanation of scripts
### application.py
The application.py file is the main script. It links to all relevant files and apps.

The script first defines a set of functions, each of which is called at a later stage.

It then initializes the scheduler and loads any existing schedules.

The script then sets the script for the webapp. By calling the /alert page, the user triggers a number of functions, including the main function: check_class_availability. This function loads a simplified version of the Pineapple website, selects the relevant date from the timetable, and then searches all entries for the teacher's name, and his class availability. The remaining functions are used to end the tasks that have been triggered with the alert, when the alert is stopped.

### Procfile
The Procfile is required to launch a Heroku app. If a user is using a different server to run their app, then the Procfile does not need to be included in the directory.

### requirements.txt
The requirements.txt file lists all libraries, which the application.py python scripts uses, referenced at the start of the script.

### index.html
The index.html script, when launched, allows users to select the teacher and the class date, for which they wish to set up an alert.
The list of teachers is passed into the html file from the application.py script.
The date selection is limited to be within the next two weeks from the date of usage.

### alert.html
The alert.html script is called when an alert has been set. It advises the user that an alert has been set, naming both the teacher and stating the date for the alert.
Users are then given the option to stop all alerts by clicking the button at the bottom of the page.

### class_status.txt
The class_status text file saves the status of the class for which an alert has been created. The application only sends an alert when the status changes from no availability to free availability. It does not continuously send alerts every 10 minutes where the class has free space. This allows a more-friendly experience. Instead, the app triggers a telegram message when the class availability status switches back from available to not available.
The class_status file saves the current status of the class.
The reason this application does not simply use a global variable for this purpose, is that the app migth get shut down, as the heroku app does - every 30min -, and would therefore loose access to the previous status. By saving the status as a text file, the application continuously has access to the latest status.
The class_status file is wiped once the alert is stopped.

### job_alert.txt
The job_alert text file is created to keep track of the API scheduler information. As for the class_status, a global variable does not work as it gets wiped when the script is restarted.
The python script saves the teacher name, class information, and unique job-id to the job_alert.txt file. This information is called when the API scheduler is relaunched, and when a user access the webpage showing the "stop-alert" message.

## Heroku settings
The heroku app runs with two buildpacks:
* **[Chrome for Testing, Python](https://buildpack-registry.s3.amazonaws.com/buildpacks/heroku-community/chrome-for-testing.tgz)** - This buildpack allows the use of Chrome to scrape the Pineapple website.
* **Python** - This buildpack enables python on heroku

We also defined two configuration variables:
* **BOT_TOKEN** - The bot token that allows the script to send a telegram blast on the chat
* **CHAT_ID** - The chat id for the telegram chat where the alerts are sent out from
