# ab-test-project
WNW streaming A/B testing project from my Applied Analytics subject at university. <br />

## Background
WNW is a streaming company which is refining their recommendation engine to provide better recommendation to customers. <br />
Better recommendations improve user engagement and importantly increase the average hours watched per user per day, a key metric used to price ads for 3rd party marketing companies. <br />
<br />
The executives at WNW want to know if the new recommendation engine algorithm is worth rolling out to all their subscribers. They have asked you to analyse the results from a recent change they made in their recommendation engine and present the results to the executive team. <br />

## The sample data
The executives are also interested in any other insights you may learn from the sample data (available to download on my ab-test-project repository). In particular, they are curious about the following: 
- Is there any bias in the data collection?
- How could any bias be corrected?
- What improvements could be made to future A/B tests?

The sample data shows the effect of an A/B test conducted to measure the effectiveness of a change to the recommendation engine used on some subscribers, but not others. The change to the recommendation went live at 1-minute past midnight on the 18th of July. <br />
Those customers who were unknowingly using the new recommendation engine to suggest what to watch next are labelled as group B, while group A was used as a control group. <br />
<br />
The data set has the following fields: <br />
- date in format dd/mm
- gender of the customer
- age of the customer
- social metric, which is a combined metric based on previous viewing habits
- number of months since the customer signed up
- demographic number
- group (A/B) where A is the control and B is the treated group
- number of hours watched in that day

The streaming_data.csv contains the dataset. <br /> 
The complete code for the project is available via the ab_testing_project.Rmd file. <br />
The final report can be viewed via https://carimo198.github.io/ab-test-project/ <br />
