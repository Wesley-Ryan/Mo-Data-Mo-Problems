# Mo’ Data... Mo’ problems…

Recently I was honored to be offered the chance to work on Blue Witness. A super cool project with a great team developing an interactive map that identifies potential instances of police use-of-force across the United States of America. The project was currently deployed with one data source and the road map called for two, which is where I was able to jump on board. But before we get started I’d like to shed a little light on the organization and some of the awesome work they’re doing;

```
Human Rights First is an independent advocacy and action organization that challenges America to live up to its ideals. We believe American leadership is essential in the global struggle for human rights, so we press the U.S. government and private companies to respect human rights and the rule of law. When they fail, we step in to demand reform, accountability and justice. Around the world, we work where we can best harness American influence to secure core freedoms. We know it is not enough to expose and protest injustice, so we create the political environment and policy solutions necessary to ensure consistent respect for human rights. Whether we are protecting refugees, combating torture, or defending persecuted minorities, we focus not on making a point, but on making a difference. For almost 40 years, we've built bipartisan coalitions and teamed up with frontline activists and lawyers to tackle global challenges that demand American leadership. Human Rights First is a non-profit, nonpartisan international human rights organization based in New York, Washington D.C., Houston, and Los Angeles.

```

### The Idea

**Now that we're all caught up, let's jump in:**

The project was scheduled to last for 4 weeks with the main objective of adding an Administration dashboard, Authentication and a new data source. With a team of frontend engineers, data scientists and _moi_ heading up the backend we headed out into the wild with the hopes of pulling relevant data from Twitter, running it through an MPL model and delivering it to our new fancy Admin dashboard.

### The Process

The steps I had set to take were as follows..

1. Analyze the current code base.. Where am I again??

2. Reconstruction the Database for the new incoming data source.

3. I ❤️ Knex.

4. Routes for Every Occasion

5. Reconstruct the Database again..

I started to get my footing in the codebase, it was a large Express app with a few routes, a Postgres database and an overall great structure. My first approach was to meet with the data science team and create an overall flow of the application. We broke this process into two parts;

- How the data would flow from the DS API to the Web API

  <img src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/DS%20FLOW.jpg" alt="dsflow" style="zoom:50%;" />

- How the data will flow throughout the Web App.

<img src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/webAppflow.jpg" alt="webAppflow" style="zoom:50%;" />

### Let's Dive In

Since the app was currently in production my first step was to deploy a development environment to avoid destroying the database just yet, I mean it’s only week one we have plenty of time for that later. I spent a bit of time getting the environment running locally, this was my first time using Docker to set up a Postgres Database (sounds like another post of it’s own , link coming soon) followed by a small dual with Heroku and I was ready to start. First step was to restructure the database, Knexjs to the rescue.

**What is Knex?** Knex.js is a "batteries included" SQL query builder for Postgres, MySQL, MariaDB, SQLite3, and Oracle designed to be flexible, portable, fun and easy to use. I started by implementing the tables for the new data source and eventually the user profiles for the admin account.

<img src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/createTables.png" alt="migration" style="zoom:50%;" />

**On to the seeds**

Now that I have a table set up, I wanted to put some data into it. So I made an array of objects that match the structure of the migration created above.

<img src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/seed2.png" alt="seeds" style="zoom:50%;" />

A few npm one liners and my Postgres database was up and running with fake data, for now. My next step was to start building the model for the new Twitter incidents.

<img src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/model.png" alt="model" style="zoom:50%;" />

getLastID turned out to be my faviorte, I had no idea at the time but this little guy here would be directly responible for POST success and nightly updates.

### Routes for Every Occasion

The basic purpose of my API was to get data from two outside sources, restructure and store the data, and finally serve it up to the frontend. To say the least basic CRUD functions were needed…

<img src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/routes.png" alt="routes" style="zoom:50%;" />

Remember that little guy we can always call upon to give us the last known ID in our database? Well he sure came in handy when writing our POST endpoint. Since we were not creating a new unique ID we had to find a way to be able to assign an ID when an incident was created from the admin dashboard. To resolve this issue I wrote a piece of middleware that will find the lastknown id in the database, increase it by 1 and attach it to the object the admin was POSTING from the dashboard.

<img src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/addIDtoPost.png" alt="addID" style="zoom:50%;" />

Now the api was up and running, deployed on our testing environment with seeded data I felt pretty good. Time for our stakeholder meeting…

### 99 problems and Data turned out to be all of them…

At our stakeholder meeting it was brought to attention by one of the data science engineers the model used to categorize the data of the current incidents in production was “immature”,we could say. We might have also said all of the current data was miscatorgized and meaningless, but let’s stick with immature. After a few umms and awws… a little coffee and some more discussion the solution was to run the production data through the new model. We would Recategorize all the data according to this model, update the data disclaimers and thus the project would move forward.

<img width='100%' style="width:100%" src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/Fine.gif ">

### Back to the drawing board

I restructured the database to accommodate the new categorized, nuked and rebuilt and was up and running with the new recategorized incidents. At last the data was flowing, the API was stable and I was about to pat myself on the back with a job well done. The next morning the new datasource API was ready and I was excited to start pulling real data into my API, I made a test call using Insomnia and…

Wait… What’s this?? The schema is nothing like I was told, this shape is not going to fit into my database. Let me reach out to the team to clarify the shape of the object.

<img width='100%' style="width:100%" src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/what.gif">

After a quick chat with the Frontend team I was able to pull apart the data and restructure the object to meet the requirements of an incident. Finally our API was up, running with Correct data and yet there was still something missing. Oh yeah updates… We should probably try to pull data in every once in awhile right?

### Node-Cron

What is it? Well the node-cron module is tiny task scheduler in pure JavaScript for node.js Sounds perfectly spledned for my use case, quick dive into the docs and simple enough a cron job is born.

<img width='100%' style="width:100%" src="https://github.com/Wesley-Ryan/Mo-Data-Mo-Problems/blob/new/assets/cron.png">

In the function above we are scheduling two tasks to be ran at the 23rd hour of every day. I evenetually removed the console logs, before and loads of testing was done to ensure the app wouldn't crash loading 200 to 500 at one point I think it was up to 3000 incidents loaded over night. At this point we had a fully functional API, a groovin PG database, Authentication was done(I know I didn't touchbase on this but really I need something to write about next time) and nightly updates to offically meet our MVP.

### Closing Statements

Working with an amazing team and a wonderful organization, I feel truly honored to have been a part of this project. There is still some room for improvements, and this will continue for another round with some new lucky developers. But in the end we met our MVP, completed our roadmap for this round, put out a small fire, and ended up with a really great release.
