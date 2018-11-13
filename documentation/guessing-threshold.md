# Guessing Threshold

Using the time spent to solve each item, we can set a threshold for which we're pretty sure that if the student spent a shorter time than the threshold, they were guessing.

## Recalculating

The calculation will go a little something like this...
```matlab
binranges = 0:100;
[bincounts]=hist(data_table.time_to_solve,binranges);

% determine the location of the second peak which should occur sometime
% between 8-20 s
[peak2 peak2_ind] = max(bincounts(9:21));
peak2_ind = peak2_ind + 8;
% make peak2 index 10s or higher
if peak2_ind < 11
  peak2_ind = 11;
end

% find the response time that corresponds to the min frequency between
% 3s and peak2_index s
min1 = min(bincounts(4:peak2_ind));
min1_inds = find(bincounts(4:peak2_ind)==min1) + 3;
min1_ind = max(min1_inds);

% now look to see if there are any frequencies that are close (but to the right)
total_responses = length(data_table.time_to_solve);
peak1_ind = 4; % 3 seconds
threshold_ind = min(find(bincounts(peak1_ind:peak2_ind) <= (min1 + ceil(total_responses*.0025))))  + (peak1_ind - 1);

% use this is picking a slightly larger threshold value
%threshold_ind = max(find(bincounts(min1_ind:peak2_ind) <= (min1 + ceil(total_responses * .0025)))) + (min1_ind - 1);
```

1.  Calculate the number of attempts that have time_to_solve between 0 and 100 seconds for each second. (You probably only need to go up to ~20 s). Call this bincounts.  If we plotted bincounts against response time then we would get a histogram (w/ hopefully 2 peaks).

2.  Determine the time when the second peak occurs, which should be sometime between 8 and 20 seconds. Call this value peak2_index.

3.  Find the local minima of bincounts that occurs between 3 seconds and peak2_index seconds.  Call this min1

* In the end I want to pick a larger threshold if there is another column in bincounts that is close in time and only slightly larger than min1.

4.  The “slightly” larger value that I think is acceptable is within 0.25% of min1.  Calculate the value that is “slightly" larger = min1 + ceil(total_responses * .0025)

5.  If there is a time that is ~~larger~~ less than the time specified by min1 and less than peak2_index that corresponds to a number of attempts < “slightly” larger value then pick this as the threshold.
Otherwise, use the time that corresponds with min1.

NEATO GRAPHICS :)

example of finding "conservative" thresholds:
![example_thresholds_conservative](https://cloud.githubusercontent.com/assets/2788801/5248426/0bbe4a20-7949-11e4-8d31-5b1a5d331b07.png)

example of finding "slightly" longer thresholds:
![example_thresholds](https://cloud.githubusercontent.com/assets/159591/5134322/01947d42-70db-11e4-8d15-60c5e062c257.png)
