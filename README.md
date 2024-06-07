## Adbot Ad Engagement Forecasting Challenge

### Description

The objective of this challenge is to accurately predict the number of clicks a client’s ad receives, one and two weeks into the future.

### Data available

The data provided contains the daily ad records for 185 clients from the Adbot platform. Each record contains information related to the ads including the cost, number of impressions, as well as the number of keywords used in the ads.
- Train.csv
- SampleSubmission.csv


### Approach

By analyzing the submission file, we can observe that there are 96 IDs whose first forecast date is 2024-02-21. Since there are no IDs with a later date than this, we can infer that the host extracted the clients' historical data around February 21, 2024.

Now that we know that the present day is around 2024-02-21, we can assume that the other 89 IDs were inactive at the time of the extraction. 

This leaves us with two groups:

  i. Active Clients.
  
  ii. Inactive Clients.

Analyzing the Inactive Clients, I decided to break them into two subgroups: (i) those who went inactive in 2023 or earlier, and (ii) those who went inactive in 2024.

```
print(len(ids_with_2024_02_21)) -> 96

print(len(inactive_ids_with_2024)) -> 33

print(len(inactive_ids_with_2023)) -> 56
```


I wrote a code to find the IDs that have a gap greater than 8 days between their last historical record and the date requested in the submission file. There are 94 IDs with such a gap.

So far, we know that our data is not homogeneous at all. We have inactive clients, active clients, and clients from both groups that have data gaps leading up to the submission file.

It would be impossible to know what to do if we didn't have the exact same situation in the data given to us. In the provided data, we have several IDs that went inactive for a period of time but suddenly came back and started publishing ads again. We can learn from these IDs to come up with a solution for the forecast period.

I will show one of these types of clients:


We can see that the days leading to inactivity are marked by a constant decrease in the metric features. Also, when the client decides to return to the platform, there is no "adaptation" period. The metrics, instead, behave as if the inactivity never happened.

While analyzing these "gap" clients, I decided on my strategy:
- For inactive clients, the forecast will show a decreasing rate in the metrics.
- Clients with a gap between the historical data and the submission date will have their dates updated to be closer to the submission date.

### Forecasting Phase
For the active clients, I realized it was important to maintain their historical patterns because, well, they are still active. So I wrote a code to analyze every feature for every ID in this group and classify whether a given feature for a given ID is stationary or not. If yes, then I forecast all the features using SARIMA; otherwise, a moving average was used (from baseline)

For the inactive clients, the descriptive ad features are forecasted using a moving average, and the metric features use a decreasing rate. We apply different decreasing rates for Inactives_2023 and Inactives_2024 because the inactive group of 2024 shows higher metrics at the beginning of the year, so consequently, they will also take a longer time to decrease.

After the script is run, I concatenate all the groups to form my final dataframe. The final data transformation updates the last recorded dates for the "gap IDs" clients.


Finally, a CatBoost regressor is trained on the entire dataset and then used to make the predictions.








