---
title: Shutting Down RetirementPlan.io
description: RetirementPlan.io has been officially shut down.
tags:
  - finance
  - web development
  - retirementplan.io
image: /assets/article_images/rpio_screenshots/3_portfolio_selection.png
---

All told, I have been working on RetirementPlan.io in some form or another for a few years. Yesterday, I shut down the website.

I launched this application as a true startup corporation, and was even featured as "Startup of the Week" in the Calgary Herald. Unfortunately the Canadian Financial Regulations are not quite ready for 100% algorithmic financial advice (unlike in the U.S.) so I had to shelve this project.  Good learning experience in any case!

---

The first version of this software was my first real web application.  It started as a means to learn [Ruby on Rails][Rails] and scratch a personal itch related to finance, retirement planning and statistics.  It grew from there into a full-fledged customer-facing application, incorporating individual user input, modern financial research and portfolio tracking.

---

The math side of the application was fairly complex... I ended up using all of the following in some fashion:

- Inverse Normal & Student's T distributions
- Cholesky decompositions
- Confidence Intervals
- Monte Carlo statistical simulations
- Regression and analysis of historical ETF returns
- Modern Portfolio Theory (portfolio variance & expected return)
- Reverse portfolio optimization

---

On the technology front, the app went through a few iterations:

- Standard server-rendered Rails app
- Rails API back-end, Ember.js front-end
- Pulled out individual services to use the correct tool (monte carlo simulations / financial analysis)

A broad list of technologies that were used at some point:

- Ruby/Rails
- Ember.js
- Postgres
- Memcached
- Redis
- Sidekiq
- Python/Flask
- Numpy/Pandas
- Golang
- Heroku/OpenShift
- And others....

---

On the marketing front, I feel I did a very good job on the numerical side, but less so on the generic PR/outreach front.  I had baked together Google Analytics / conversion tracking / event tracking / transactional email / etc.

On the PR front, I did managed to get covered by the [Calgary Herald](http://blogs.calgaryherald.com/2014/07/21/startup-of-the-week-retirementplan-io/) which drove significant traffic and signups, but nothing much more exciting than that.

---

There is not much of a story in terms of the reasons the site was shut down. The Ontario Securities Commission released (Q3'14) an opinion statement regarding trends in registration applications. They specifically called out the fact that Canadian Regulators were not ready for purely algorithm-driven financial advice.

I had put significant research and effort into trying to provide industry-standard tooling into the hands of every Canadian who was interested, but was not particularly interested in building up a "standard" retirement advice practice, which is what the regulations currently require.

The worst part is that I know for a fact there is a non-zero number of "financial advisors" charging exorbitant rates for nothing more than a spreadsheet and super-basic math that gets taught in first-year finance classes. It is a shame really.

---

All told, RetirementPlan.io was not the overwhelming success that I had hoped for (and I'm convinced the potential still exists if regulations change), but I'd say it was a very good learning experience - sharpened my financial skills, taught me about some of the ins-and-outs of incorporating in Canada, and I built a ton of programming/web development knowledge.

---

Some screenshots:

![User Preferences](/assets/article_images/rpio_screenshots/0_user_preferences.png) 
![User Profile](/assets/article_images/rpio_screenshots/0_user_profile.png)
![Dashboard](/assets/article_images/rpio_screenshots/1_dashboard.png) 
![Risk Tolerance Questionnaire](/assets/article_images/rpio_screenshots/2_risk_tolerance_questionnaire.png)
![Portfolio Selection](/assets/article_images/rpio_screenshots/3_portfolio_selection.png) 
![Simulation Expenses](/assets/article_images/rpio_screenshots/4_simulation_expenses.png) 
![Simulation Parameters](/assets/article_images/rpio_screenshots/4_simulation_parameters.png) 
![Select ETFs](/assets/article_images/rpio_screenshots/5_select_ETFs.png) 
![Set up a Tracked Portfolio](/assets/article_images/rpio_screenshots/7_set_up_tracked_portfolio.png) 
![Portfolio Status](/assets/article_images/rpio_screenshots/8_portfolio_status.png) 
