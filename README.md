# Project Scenario 🎩
In this practice project, you will:
- Initialize your log file
- Write a Bash script to download, extract, and load raw data into a report
- Add some basic analytics to your report
- Schedule your report to update daily
- Measure and report on historical forecasting accuracy

# Reach/Follow me on <br>
<p align="left">
  <a href="https://www.linkedin.com/in/mohamed-fawzy-936b661b8/" target="_blank" rel="noreferrer"> <img src="https://img.icons8.com/fluency/2x/linkedin.png" alt="linkedIn" width="50" height="50"/> </a>&nbsp&nbsp
  <a href="mailto:fwzymohamed90@gmail.com" target="_blank" rel="noreferrer"> <img src="https://img.icons8.com/fluency/2x/google-logo.png" alt="googleEmail" width="50" height="50"/> </a>&nbsp&nbsp
  <a href="https://www.facebook.com/mohamed.fwzy.14" target="_blank" rel="noreferrer"> <img src="https://cdn.iconscout.com/icon/free/png-256/facebook-262-721949.png" alt="facebook" width="50" height="50"/> </a>
</p>
<br>

# Directions 🗺
### Exercise 1 - Initialize your weather report log file
- 1.1 Create a text file called `rx_poc.log`
```bash
touch rx_poc.log
```
- 1.2 Add a header to your weather report
```bash
echo -e "year\tmonth\tday\thour\tobs_tmp\tfc_temp">rx_poc.log
```

### Exercise 2 - Download the raw weather data
- 2.1. Create a text file called `rx_poc.sh` and make it an executable Bash script
```bash
touch rx_poc.sh
```
Make your script executable by running the following in the terminal:
```bash 
#! /bin/bash
```
OR

```bash 
chmod u+x rx_poc.sh
```
- 2.2 Edit `rx_poc.sh` to download today's weather report from wttr.in
```bash 
weather_report=raw_data_$(date +%Y%m%d)
```

- 2.3 Download the wttr.in weather report for Casablanca and save it with the filename you created
```bash
city=Casablanca
curl wttr.in/$city?format="%t\n%T\n" --output $weather_report
```

### Exercise 3 - Extract and load the required data
- 3.1. Edit `rx_poc.sh` to extract the required data from the raw data file and assign them to variables `obs_tmp` and `fc_temp`
```bash 
grep °C $weather_report > temperatures.txt
```
- 3.1.1. Extract the current temperature, and store it in a shell variable called
```bash 
obs_temp=$(head -1 $weather_report)
echo "Current Temperature: $obs_temp"
```
- 3.1.2. Extract tomorrow's temperature forecast for noon, and store it in a shell variable called `fc_tmp`
```bash 
fc_temp=$(head -4 $weather_report | tail -1 | grep -oE '[0-9]+' | head -1)
echo "Forecast for Noon Tomorrow: $fc_temp"
```
- 3.2. Store the current hour, day, month, and year in corresponding shell variables
```bash 
hour=$(TZ='Morocco/Casablanca' date -u +%H) 
day=$(TZ='Morocco/Casablanca' date -u +%d) 
month=$(TZ='Morocco/Casablanca' date +%m)
year=$(TZ='Morocco/Casablanca' date +%Y)
```

- 3.3. Merge the fields into a tab-delimited record, corresponding to a single row in Table 1
```bash 
record=$(echo -e "$year\t$month\t$day\t$obs_tmp\t$fc_temp")
echo $record>>rx_poc.log
```
### Exercise 4 - Schedule your Bash script `rx_poc.sh` to run every day at noon local time
- 4.1. Determine what time of day to run your script
```bash 
date
date -u
```

- 4.2 Create a cron job that runs your script, and this at the end of the file `0 8 * * * /home/project/rx_poc.sh`
```bash 
crontab -e
```

### Exercise 5 - Create a script to report historical forecasting accuracy
- To begin, create a tab-delimited file called historical_fc_accuracy.tsv using the following code to insert a header of column names:
```bash 
echo -e "year\tmonth\tday\tobs_tmp\tfc_temp\taccuracy\taccuracy_range" > historical_fc_accuracy.tsv
```
- 5.1. Determine the difference between today's forecasted and actual temperatures
- 5.1.1. Extract the forecasted and observed temperatures for today and store them in variables
```bash 
yesterday_fc=$(tail -2 rx_poc.log | head -1 | cut -d " " -f5)
```

- 5.1.2. Calculate the forecast accuracy
```bash 
today_temp=$(tail -1 rx_poc.log | cut -d " " -f4)
accuracy=$(($yesterday_fc-$today_temp))
```

- 5.2. Assign a label to each forecast based on its accuracy


|       accuracy range                | 🔰 accuracy label  
| -------------------------- | :----------------:| 
| +/- 1 deg            |         Excellent         |    
| +/- 2 deg            |         Good         |    
| +/- 3 deg             |         Fair         |    
| +/- 4 deg   |         poor         |  

```bash
if [ -1 -le $accuracy ] && [ $accuracy -le 1 ]
then
   accuracy_range=excellent
elif [ -2 -le $accuracy ] && [ $accuracy -le 2 ]
then
    accuracy_range=good
elif [ -3 -le $accuracy ] && [ $accuracy -le 3 ]
then
    accuracy_range=fair
else
    accuracy_range=poor
fi

echo "Forecast accuracy is $accuracy"
```

- 5.3. Append a record to your historical forecast accuracy file.
```bash 
row=$(tail -1 rx_poc.log)
year=$( echo $row | cut -d " " -f1)
month=$( echo $row | cut -d " " -f2)
day=$( echo $row | cut -d " " -f3)
echo -e "$year\t$month\t$day\t$today_temp\t$yesterday_fc\t$accuracy\t$accuracy_range" >> historical_fc_accuracy.tsv
```

- 5.4. Full solution for handling a single day
```bash
#! /bin/bash

yesterday_fc=$(tail -2 rx_poc.log | head -1 | cut -d " " -f5)
today_temp=$(tail -1 rx_poc.log | cut -d " " -f4)
accuracy=$(($yesterday_fc-$today_temp))

echo "accuracy is $accuracy"

if [ -1 -le $accuracy ] && [ $accuracy -le 1 ]
then
           accuracy_range=excellent
elif [ -2 -le $accuracy ] && [ $accuracy -le 2 ]
   then
               accuracy_range=good
       elif [ -3 -le $accuracy ] && [ $accuracy -le 3 ]
       then
                   accuracy_range=fair
           else
                       accuracy_range=poor
fi

echo "Forecast accuracy is $accuracy_range"

row=$(tail -1 rx_poc.log)
year=$( echo $row | cut -d " " -f1)
month=$( echo $row | cut -d " " -f2)
day=$( echo $row | cut -d " " -f3)
echo -e "$year\t$month\t$day\t$today_temp\t$yesterday_fc\t$accuracy\t$accuracy_range" >> historical_fc_accuracy.tsv
```

### Exercise 6 - Create a script to report weekly statistics of historical forecasting accuracy
- 6.1. Download the synthetic historical forecasting accuracy dataset
- 6.2. Load the historical accuracies into an array covering the last week of data
- 6.3. Display the minimum and maximum absolute forecasting errors for the week
  #### Your final shell script should now look something like the following:
```bash
#!/bin/bash

echo $(tail -7 synthetic_historical_fc_accuracy.tsv  | cut -f6) > scratch.txt

week_fc=($(echo $(cat scratch.txt)))

# validate result:
for i in {0..6}; do
    echo ${week_fc[$i]}
done

for i in {0..6}; do
  if [[ ${week_fc[$i]} < 0 ]]
  then
    week_fc[$i]=$(((-1)*week_fc[$i]))
  fi
  # validate result:
  echo ${week_fc[$i]}
done

minimum=${week_fc[1]}
maximum=${week_fc[1]}
for item in ${week_fc[@]}; do
   if [[ $minimum > $item ]]
   then
     minimum=$item
   fi
   if [[ $maximum < $item ]]
   then
     maximum=$item
   fi
done

echo "minimum ebsolute error = $minimum"
echo "maximum absolute error = $maximum"
```

# Contributing 📝
Contributions are welcome! Please open an issue or pull request for any changes or improvements.








