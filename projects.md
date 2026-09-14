# Projects
This section documents my data science projects, research questions, and data stories I create throughout the semesters.
---
## Project 1
Research project planning
The topic of my project at the broadest level is: 
The everyday financial challenges faced by average U.S. citizens, especially overspending and budgeting difficulties.

Two possible specific and answerable research questions I might ask are: 
	To what extent does making more than three impulse purchases per week increase the average monthly overspending amount among U.S. adults aged 18–35?

	Does using a budgeting app at least five days per week reduce the likelihood of exceeding one’s monthly budget by more than $100?


My key variables are: 
RQ1-
	Monthly overspending amount
	Frequency of impulse purchases
	Age (restricted to 18–35)
	Income level (to control for differences)


Here’s how I’m conceptualizing (defining) each of my variables: 
Monthly overspending amount: The difference between a person’s planned monthly budget and their actual spending. Overspending occurs when actual spending exceeds the planned budget.
Frequency of impulse purchases: The number of unplanned purchases made per week. An impulse purchase is defined as any item bought spontaneously without prior intention (e.g., snacks, small electronics, clothing, convenience items).
 Age: The respondent’s age, limited to individuals between 18 and 35 years old.
 Income level: The respondent’s annual income, grouped into brackets (e.g., <$30k, $30–60k, >$60k) to help control for spending differences.



Here’s how I’m operationalizing (measuring) each of my variables: 
	Monthly overspending amount: Numeric variable measured in dollars. Calculated as:
"Overspending"="Actual Monthly Spending"-"Planned Monthly Budget" 
	Frequency of impulse purchases: Count variable measured as the number of impulse purchases per week, self reported through survey questions such as: “How many unplanned purchases did you make last week?”
	Age: Numeric variable collected directly, then filtered to include only respondents aged 18–35.
	Income level: Categorical variable based on self reported annual income bracket.



The data I ideally need is: 
	Survey responses capturing weekly impulse purchases
	Monthly budget vs. actual spending data
	Demographic information (age, income)
	Optional: transaction level purchase data if available



Where I’m looking for possible access to this data: 
	Bureau of Labor Statistics Consumer Expenditure Survey- https://www.bls.gov/cex/
	Kaggle datasets on personal finance and spending behavior
	UNC Charlotte library databases- https://library.charlotte.edu/research-write/databases-
	Self created Qualtrics or Google Forms survey distributed to students
	Pew Research datasets on consumer habits (if relevant)- http://pewresearch.org/datasets/
	


Two visualizations I might ideally make are: 
  Scatter plot
	X axis: Number of impulse purchases per week
	Y axis: Monthly overspending amount
	Purpose: Show whether higher impulse purchases correlate with higher overspending.
Box and whisker plot
	Groups:
	Group 1: People making ≤3 impulse purchases/week
	Group 2: People making >3 impulse purchases/week
	Y axis: Monthly overspending amount
	Purpose: Compare overspending distributions between low impulse and high impulse buyers.




Questions I still have are: 
 Should I control additional variables like education level or employment status?
 Will self reported impulse purchase data be reliable enough, or should I attempt to collect transaction logs?
 Should I analyze overspending as a continuous variable or categorize it (e.g., overspent by $0–50, $50–100, >$100)?


 Code coming soon



