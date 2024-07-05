## Data Wrangling

### Turning Zoom VTT files into insightful visualisation

#### *A Data Wrangling Project*

The COVID-19 pandemic revolutionised education by accelerating the adoption of digital tools, with Zoom emerging as a pivotal platform. Zoom enabled educators to conduct live lectures, facilitate interactive discussions, and maintain a sense of community despite physical distancing. This shift not only allowed for the immediate continuation of education but also paved the way for a more flexible, hybrid model of learning that is likely to persist beyond the pandemic.

Class participation in Zoom lectures has transformed traditional educational dynamics, offering both challenges and opportunities. Since speaking, debating and contributing ideas in an adult-learning environment is important to developing critical thinking and collaborative skills, it becomes crucial to measure the level of class participation in Zoom. So how does one determine the level of class participation of a student in a Zoom environment?

An auto-generated transcript from a Zoom class looks like this:

    WEBVTT
    
    1
    00:01:45.690 --> 00:01:46.260
    INSTRUCTOR: hey.
    
    2
    00:01:47.550 --> 00:01:50.430
    INSTRUCTOR: Everyone knows the drill sitting on the cameras okay very good.
    
    3
    00:01:51.930 --> 00:01:52.980
    INSTRUCTOR: Okay.
    
    4
    00:02:04.830 --> 00:02:07.500
    INSTRUCTOR: Okay we're going to take the picture.
    
    5
    00:02:12.360 --> 00:02:14.280
    INSTRUCTOR: Yes, checking so okay

<br />
### and now we wrangle, with a Python of course 

It was my first experience with Python programming and I quickly learned that its fussiness over syntax is like having a grammar-loving parrot perched on your shoulder, constantly squawking about indentation and backslashes. But to deal with the above data format, Python is the program.

#### missing speakers?
From the raw ‘vtt’ table, it was observed that fifty-two lines of the transcript did not contain a ‘RegName’. Since the duration of these utterances could affect the length of time spoken by the instructor or any of the students, the ownership of utterances with missing speakers had to be reasonably assigned. In addition, the ‘RegName’ needed to be automatically assigned by the Python program to allow it to be reusable on other ‘vtt’ files.

Random samples were selected to study the preceding and succeeding utterances with missing speakers. It could be reasonably established that an utterance without a ‘RegName’ would likely come from the preceding speaker. Therefore, when a ‘NameUtt’ did not contain a ‘RegName’, the program would assign the previous ‘RegName’ to that particular line of utterance. 


    lookingFor = 'INIT'
    for i,current_line in enumerate(all_lines):
        if lookingFor == 'INIT':
            mo = INIT_regex.search(current_line)
            if mo != None:
                lookingFor = 'SNo'
            continue
        elif lookingFor == 'SNo':
            mo = SNo_regex.search(current_line)
            if mo != None:
                SNo = mo[1]
                lookingFor = 'Time'
        elif lookingFor == 'Time':
            mo = Time_regex.search(current_line)
            if mo != None:
                TimeFrom = mo[1]
                TimeTo = mo[2]
                lookingFor = 'NameUtt'
        elif lookingFor == 'NameUtt':
            mo = NameUtt_regex.search(current_line)
            if mo != None:
                RegName = mo[1]
                Utterance = mo[2]
                previous_RegName = RegName 
                lookingFor = 'SNo'
            else:
                lookingFor == 'UttOnly'
                mo = UttOnly_regex.search(current_line)
                if mo != None:
                    RegName = previous_RegName # assign preceding speaker when there is no Regname
                    Utterance = mo[1]
                    lookingFor = 'SNo'
            df.loc[df.size] = (SNo, TimeFrom, TimeTo, RegName, Utterance)

    df.reset_index(inplace=True, drop=True)

After much wrangling, I generated a table with five columns: SNo, TimeFrom, TimeTo, RegName and Utterance. This data-driven approach ensured that all students who were actively involved in classes were measured appropriately.

Results from data wrangling:

    SNo	TimeFrom    TimeTo	RegName	        Utterance
    1	01:45.7	    01:46.3	INSTRUCTOR	hey.
    2	01:47.5	    01:50.4	INSTRUCTOR	Everyone knows the drill sitting on the cameras okay very good.
    3	01:51.9	    01:53.0	INSTRUCTOR	Okay.
    4	02:04.8	    02:07.5	INSTRUCTOR	Okay we're going to take the picture.
    5	02:12.4	    02:14.3	INSTRUCTOR	Yes, checking so okay.

<br />
### right tool for the job 

There were some data cleaning activities to perform to get the TimeFrom and TimeTo into a time string format in milliseconds (for summing purposes later on). Although Python is capable of doing that, using SQL just makes life easier—like using a wine opener for wine corks instead of a screwdriver.

    CREATE TABLE IF NOT EXISTS vttclean (
        	     SNo INT,
            TimeFrom VARCHAR(12),
              TimeTo VARCHAR(12),
             RegName VARCHAR(15),
           Utterance VARCHAR(300),
        milliseconds INT);
    
    INSERT INTO vttclean (SNo, TimeFrom, TimeTo, RegName, Utterance, milliseconds)
    SELECT
        SNo, TimeFrom, TimeTo, RegName, Utterance,
        (TIME_TO_SEC(STR_TO_DATE(TimeTo,   '%H:%i:%s.%f')) * 1000 + RIGHT(TimeTo,   3)) -
        (TIME_TO_SEC(STR_TO_DATE(TimeFrom, '%H:%i:%s.%f')) * 1000 + RIGHT(TimeFrom, 3)) AS milliseconds
    FROM vtt;

<br />
![](SQLout.png)

<br />
### time to visualise

Once the time formats were in order, the R program was used to plot a horizontal bar chart to show which student babbled the most in class. I’m not sure if marks for class participation were awarded in this manner, but my work for this exercise is done. 😎

    # plot visualisation
    ggplot(filtered.d, aes(x = total.milliseconds, y = RegName, 
                           fill = as.numeric(total.milliseconds))) +
      geom_segment(aes(x = 0, xend = total.milliseconds, y = RegName, 
                       yend = RegName), colour = '#B81840', size = 7) +
      geom_point(size = 7, color = '#81B2D4') +
      geom_point(size = 1.5) +
      geom_vline(xintercept = 0, linetype = "solid") +
      geom_text(aes(label = scales::comma(total.milliseconds)), 
                position = position_stack(vjust = 1), size = 3, 
                hjust = -0.6) + 
      labs(x = "Air Time (milliseconds)", y = "",
           title = "Total Airtime of Students (in milliseconds)") +
      scale_x_continuous(breaks = seq(0, 300000, by = 50000), 
                         labels = scales::comma, 
                         limits = c(0,300000)) +
      theme_minimal() +
      guides(fill = FALSE) +
      theme(panel.border = element_blank(),
            plot.title = element_text(hjust = 0.5, size = 20, 
                                      family = 'serif'),
            panel.grid.major.y = element_blank(),
            panel.grid.minor.y = element_blank(),
            axis.text.y = element_text(face = 'bold', size = 12, 
                                       family = 'serif'),
            axis.title = element_text(family = 'serif'))

![Rplot03](https://github.com/KenYeoKP/mystuff/assets/167163077/223924cf-1faf-41d6-a972-42ebd9d08b28)

<a style="font-weight:bold" href="https://KenYeoKP.github.io">Return to posts and repositories</a>
