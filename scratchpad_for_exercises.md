Person1 
Anyone a pandas wizard that can help me with a feature engineering transformation? I'm trying to expand a column into 3 different ones but fill in 'NA' when a condition isn't met... :thread: (edited) 
8 replies

The data looks like this:
 | date       | time     | OriginalTitle          |   sc-status | device   |   NodeID | anon_ip      | anon_cs-user-agent   |
        |:-----------|:---------|:-----------------------|------------:|:---------|---------:|:-------------|:---------------------|
        | 2021-09-29 | 23:57:36 | S06 E05 - SeriesTitle0 |         200 | Roku     | 10294754 | 281.60.67.0  | user-agent0          |
        | 2021-09-30 | 07:07:16 | S03 E04 - SeriesTitle0 |         206 | Roku     | 10294350 | 42.41.600.0  | user-agent1          |
        | 2021-09-30 | 07:18:15 | S03 E04 - SeriesTitle0 |         206 | Roku     | 10294350 | 42.41.600.0  | user-agent1          |
        | 2021-09-30 | 02:51:05 | FilmTitle63            |         200 | Roku     | 10233175 | 943.70.584.0 | user-agent2          |

And I'm trying to take any type of TV show and expand it into the 'season', 'episode' and 'series title' based off the feature 'S03 E04 - SeriesTitle0' .... Else this value will correspond to a Film and that's where I'd want to fill in nulls.

Person2  
If I'm understanding you correctly, you could use str.split on Original tile to split on the hyphen and then a little regex to check if the first part of the split matches the pattern S(some numbers) E(some numbers). Then just use the split again on spaces to get your season #, episode #, and title. This could be turned into a function that you .apply to the column and return null if it doesn't match the regex.

Person3 
You could do something like this:
In [1]: import pandas as pd         

In [2]: df = pd.DataFrame() 

In [3]: df["title"] = [ 
   ...:     "S06 E05 - SeriesTitle0", 
   ...:     "S03 E04 - SeriesTitle0", 
   ...:     "S03 E04 - SeriesTitle0", 
   ...:     "FilmTitle63", 
   ...: ]         

In [4]: df["is_tv_show"] = df.title.str.match(r"^S\d+\sE\d+\s\-\s")     

In [5]: tv_shows = df[df.is_tv_show]         

In [6]: tv_shows      
Out[6]: 
                    title  is_tv_show
0  S06 E05 - SeriesTitle0        True
1  S03 E04 - SeriesTitle0        True
2  S03 E04 - SeriesTitle0        True

In [7]: new_features = tv_shows.title.str.extract( 
   ...:     r"^S(?P<season>\d+)\sE(?P<episode>\d+)\s\-\s(?P<series_title>.*)" 
   ...: )                     

In [8]: pd.concat([tv_shows, new_features], axis=1)         
Out[8]: 
                    title  is_tv_show season episode  series_title
0  S06 E05 - SeriesTitle0        True     06      05  SeriesTitle0
1  S03 E04 - SeriesTitle0        True     03      04  SeriesTitle0
2  S03 E04 - SeriesTitle0        True     03      04  SeriesTitle0
:white_check_mark:
1
:pandasmart:
2


Person2  
Dang, I was just about to say that using just regex from the beginning would be simpler.

Person2  
@Person3  I don't think I have seen str.extract before, but that is nice.

Person3 
Ya, and if you name the capture groups it turns them into nice named columns

Person1  
So when I tried this, I get an issue. I forgot to mention I already cleaned the title by taking out the hyphen and the tricky part is that some of the 'titles' are values like 'FilmTitle11'.
What I've tried:
df2[['Season', 'Episode', 'Series']] = df2.OriginalTitle.str.split(' ', expand=True)
Error:
ValueError: Columns must be same length as key
Then
df2_testexpand = df2.OriginalTitle.str.split(' ', expand=True)
df2_testexpand.sample(100)
Gives me an output like this:
406818  S05 E11     SeriesTitle0
1608089 S01 E17     SeriesTitle0
1435138 FilmTitle114    None    None    None
2352835 S01 E22     SeriesTitle0

Person3 
Ya I would expect an error w/ the str split method if you applied it across the whole dataframe; I’d suggest splitting into two seperate dataframes of movies and tv shows first with that approach, or if you do it with a regex and .str.extract you will just get back NAs where the regex doesn’t match