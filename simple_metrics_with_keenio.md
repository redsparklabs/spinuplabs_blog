[Keen.io Logo](https://keen_web_static.s3.amazonaws.com/img/keen_io_logo_rgb_2x.png)

We all know how important it is to track the relevant metrics for our startup.  There are numerous startups out there competing to fill this need.  I have looked at many of them and they are either hard to integrate, or they have specific metrics which they have determined to be "valid" metrics.

When building a startup you go through various stages and your important metrics change as you progress through the growth stages.  What may start out as a vanity metric soon becomes an important metric for *your* business.

For a client of ours, we are implementing a tracking system to track multiple specific events.  I have been evaluating different options for tracking these events we care about and decided to give [Keen.io](http://www.keen.io) a test run.

I wrote up a basic little rails app which has a pretty simple structure.  The world has people, and each individual person has multiple animals (wild, pets, etc).  Download my rails app [here](http://github.com/redsparklabs) and follow along.

  ![ERD Document](/assets/erd.jpeg)

First things first, you need to sign up for a keen.io account.  [Keen.io](http://www.keen.io)

Once you have confirmed your account and logged into your keen.io account, create a new project.

![Create New Project](/assets/create_project.jpeg)

Back on our local dev machine, we need to edit our app.  I am going to assume you already have a rails app built and already know what you want to track or have downloaded and initialized my [ demo app](http://github.com/redsparklabs).  For my particular app, I want to track when a visitor views a profile (person) or views an animal.  For your app that may be tracking when a user signs up or activates their account.

First we want to install the keen gem

    # gem install keen

or

Update your Gemfile

    gem 'keen'

then

    # bundle install

Now that we have our gem installed, we want to put in our Keen project id.  Since this was a quick and dirty demo, I simple set an environment variable in the environment/development.rb file.  For something you are going to push to production there are multiple ways of doing this better.

    ENV['KEEN_PROJECT_ID'] = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'



With that we are ready to start pushing event data to keen.  I am doing this in my controller, however you could do it in your view if you had to.

    def person
      @person = Person.find_by_id(params[:id])
      tracker = Keen.publish("view_profile", {:visitor_ip => request.remote_ip, :name => @person.fullname, :person_id => @person.id })
    end

For person, I am calling my event "view_profile" and tracking the visitor's ip and the name and id of the profile being viewed.

    def animal
      @animal = Animal.find_by_id(params[:id])
      tracker = Keen.publish("view_animal", {:visitor_ip => request.remote_ip, :name => @animal.name, :animal_id => @animal.id })
    end

For animal, I am calling the event "view_animal" and tracking the visitor's ip and the name and id of the animal being viewed.

If you have an app where a user logs in, you would want to track something like their user_id instead of the remote ip.  That would allow you to easily track your cohort metrics.

Now I can go back to my keen.io dashboard and using the event explorer, confirm keen.io is receiving events.

![Event Confirmation](/assets/event_confirm.jpeg)

Once keen.io is receiving events, you can use the API Workbench to quickly create data visualizations.  For my particular instance, I want to see a graph showing the event details for "view_profile".

![API Workbench Graphic](/assets/api_workbench_graphic.jpeg)
![API Workbench Javascript](/assets/api_workbench_javascript.jpeg)

Keen.io makes this step so simple.  They provide you with the details to easily use their [Javascript SDK](https://keen.io/docs/clients/javascript/usage-guide/), as well as the javascript to generate the graph content.

    <h1>Dashboard</h1>
    <script type="text/javascript">
      var Keen=Keen||{configure:function(a,b,c){this._pId=a;this._ak=b;this._op=c},addEvent:function(a,b,c,d){this._eq=this._eq||[];this._eq.push([a,b,c,d])},setGlobalProperties:function(a){this._gp=a},onChartsReady:function(a){this._ocrq=this._ocrq||[];this._ocrq.push(a)}};
      (function(){var a=document.createElement("script");a.type="text/javascript";a.async=!0;a.src=("https:"==document.location.protocol?"https://":"http://")+"dc8na2hxrj29i.cloudfront.net/code/keen-2.0.0-min.js";var b=document.getElementsByTagName("script")[0];b.parentNode.insertBefore(a,b)})();

      Keen.configure("xxxxxxxxxxxxxxxxx", "xxxxxxxxxxxxxxxx");

      // visualization code goes here
    </script>

    <h2>Profile Views</h2>
    <div id="profile_analytics">This will get replaced with the profile graph</div>
    <h2>Animal Views</h2>
    <div id="animal_analytics">This will get replaced with the animal graph</div>

    <script type="text/javascript">
      Keen.onChartsReady(function() {
        var profile_metric = new Keen.Metric("view_profile", {
          analysisType: "count",
          targetProperty: "name",
          groupBy: "name"
        });
        profile_metric.draw(document.getElementById("profile_analytics"));

        var animal_metric = new Keen.Metric("view_animal", {
          analysisType: "count",
          targetProperty: "name",
          groupBy: "name"
        });
        animal_metric.draw(document.getElementById("animal_analytics"));
      });
    </script>

And voila, you have a nice looking chart for your metrics.

![metrics dashboard screenshot](/assets/metric_dashboard.jpeg)

Next time I will cover, creating a more full-featured dashboard and charts.
