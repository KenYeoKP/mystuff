## Data Wrangling

### Speak up


The COVID-19 pandemic revolutionised education by accelerating the adoption of digital tools, with Zoom emerging as a pivotal platform. Zoom enabled educators to conduct live lectures, facilitate interactive discussions, and maintain a sense of community despite physical distancing. This shift not only allowed for the immediate continuation of education but also paved the way for a more flexible, hybrid model of learning that is likely to persist beyond the pandemic.

Class participation in Zoom lectures has transformed traditional educational dynamics, offering both challenges and opportunities. Since speaking, debating and contributing ideas in an adult-learning environment are important to developing critical thinking and collaborative skills, it is crucial to measure the level of class participation in Zoom. So how does one determine the level of class participation of a student in a Zoom environment?

An auto-generated transcript from a Zoom class looks like this:


## and now we wrangle, with a Python of course
It was my first experience with Python programming and I quickly learned that its fussiness over syntax is like having a grammar-loving parrot perched on your shoulder, constantly squawking about indentation and backslashes. But to deal with the given data format, Python is the way to go. After much wrangling, I generated a SQL table with five columns: SNo, TimeFrom, TimeTo, RegName and Utterance. This data-driven approach ensured that all students who were actively involved in classes were measured appropriately.


## right tool for the job
There were some data cleaning activities to perform to get the TimeFrom and TimeTo into a time string format in milliseconds (for summing purposes later on). Although Python is capable of doing that, but using SQL makes my life easier—like using a wine opener for wine corks instead of a screwdriver.

Once the time format was in order, a horizontal bar chart was plotted to show which student babbled the most in classes. I’m not sure if marks for class participation were awarded in this manner, but my job for this exercise is done here.