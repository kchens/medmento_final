Medmento
==============
Medmento is a medication reminder and pain management app for caretakers.

With Medmento, you (the caretaker) can send your loved ones (father, mother, and grandparents) a reminder phone call to take their medication.  You can set a daily reminder, or weekly reminder at any time you choose -- down to the minute. You can also add a personalized message, so your loved ones know you care.

Perhaps just as important, you can track how much pain your loved ones feel *over time*. Our system collects patient data on a 0-9 scale and provides graphical display. Our aim is to help caretakers and their loved ones track the perceived effectiveness of medications.

Other features we may add in the future are:  alerts for when your loved one's pain spikes, email messages to the doctor, and a log how the patient feels each day (a "Twitter" for how they feel). 

##Software Architecture

![Medmento Data Flow](imgs/Medmento_Architecture.png)

[Click here for a more detailed look](http://prezi.com/g2kx3qdhe1gd/?utm_campaign=share&utm_medium=copy&rc=ex0share).

##Taking Medmento for a Test Drive
As Medmento requires Sidekiq and Redis, we decided not to host our Rails backend on Heroku. If you'd like to test out Medmento on your local environment, follow these steps (on Mac terminal).

####Setting up the Back-end

1. `git clone https://github.com/bluehawk27/medmento_final.git` to this repository onto your local environment: 
2. `rake db:create db:migrate db:seed` in the root directory.
3. Open 5 terminal tabs. Change each tab's directory to the memento root.
4. `redis server` in the first tab.
5. `be sidekiq` in the next tab.
6. `./bin/ngrok 3000` in the next tab. 
	* Copy the ngrok url 
	* Open the `.env` file
	* Paste the ngrok url into `BASE_URL`
7. `rails s` in the next tab.
8. Sign-up for Twilio.
	* Open the `.env` file
	* Copy & paste your `ACCOUNT_SID` and `AUTH_TOKEN` from Twilio into `.env
	* Open `twilio_worker.rb` and replace `CALLER_NUM` with your phone number.
9. `be clockwork config/clock.rb` in the final tab. Clockwork will now start pinging the database at a regular interval.
	* You can change how often clockwork pings the database in the `clock.rb` file. We set Clockwork to check the database every 10 seconds.

####Setting up the Front-end
In the `medmento-frontend` folder, you'll see the static files we used to interact with our Rails API. 

To run these locally, simply (1) open the `index.html` file in your browser. Then, (2) Create, Read, Update, or Delete a reminder!* See more in "How it Works".

*In `medmento.js`, you can see that the API endpoints default to `localhost:3000/*` or `medmento.herokuapp.com`. The Heroku App **does not** make a phone call because the API requires Redis and Sidekiq.


##How it works (From The Customer's Perspective)

####Front end

1. Once logged in, a caretaker clicks "Set A Reminder" button on the top left.

2. The caretaker fills in the name of the loved one, reminder date, and a custom message.

3. Click "Save". The form data is serialized and passed as a JSON object to our Rails API. Similarly, editing or deleting a reminder on  updates or destroys the corresponding event in the back end.

####Back End

4. When all services are running (check "Setting up the Back-end"), `clockwork` will check the database every 10 seconds (you can set any interval). Clockwork looks for three Time fields in each record of our Events table. 

5. When the Time fields match the current time (i.e., Time.now), Clockwork invokes our Sidekiq worker -- `TwilioWorker`. Clockwork also passes in the relevant record data to our `TwilioWorker`

6. `TwilioWorker` processes the background job into a JSON and stores the job to Redis. Redis enqueues the job. 

7. When the job time comes, Redis dequeues the job; Sidekiq pulls the data. 

8. `TwilioWorker` makes a call to the Twilio API...to call your loved one -- with your custom message and information.
