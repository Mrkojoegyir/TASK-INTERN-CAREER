INTERN-CAREER
TASK 1
import pandas as pds
import numpy as npy

youtubers_df=pds.read_csv("youtubers_df.csv")
youtubers_df

#Changing the name of the 4th column to Subscribers
youtubers_df.rename(columns={"Suscribers":"Subscribers"},inplace=True)
youtubers_df.head(0)

# Missing data
missing_data = youtubers_df.isnull().sum()
missing_data

#Replacing the missing categories with Unknown
youtubers_df["Categories"].fillna("Unknown",inplace=True)

# Outliers
outliers = youtubers_df.describe()
outliers

from scipy.stats import zscore

# Calculating the Z-scores of the numerical columns to identify outliers
numerical_cols = ['Subscribers', 'Visits', 'Likes', 'Comments']
youtubers_df[numerical_cols] = youtubers_df[numerical_cols].apply(zscore)

# Defining outliers as those entries with a z-score greater than 3 or less than -3
outliers = youtubers_df[(youtubers_df[numerical_cols] > 3).any(axis=1) | (youtubers_df[numerical_cols] < -3).any(axis=1)]
outliers

#Dropping the links column
youtubers_df.drop(columns="Links",inplace=True)
youtubers_df.head(0)

#finding duplicates in the data
dup=youtubers_df["Username"].duplicated().sum()
dup

#removing the 6 duplicates found in the dataset
youtubers_df.drop_duplicates(subset=["Username"],inplace=True)
dup=youtubers_df["Username"].duplicated().sum()
dup

#  Trend Analysis

import seaborn as sns
import matplotlib.pyplot as plt

# Identifying the most popular categories
popular_categories = youtubers_df['Categories'].value_counts().head(10)
print('Most popular categories:\n', popular_categories)

# Plot the most popular categories
plt.figure(figsize=(12, 6))
sns.barplot(x=popular_categories.values, y=popular_categories.index)
plt.title('Top 10 Most Popular Categories among YouTube Channels')
plt.xlabel('Number of Channels')
plt.ylabel('Categories')
plt.show()

# Correlation between subscribers and likes/comments
subscribers_likes_comments_corr = youtubers_df[['Subscribers', 'Likes', 'Comments']].corr()
print('Correlation matrix between subscribers, likes, and comments:\n', subscribers_likes_comments_corr)

# Plot the correlation matrix
sns.heatmap(subscribers_likes_comments_corr, annot=True, fmt='.2f', cmap='coolwarm')
plt.title('Correlation Matrix')
plt.show()

#   Audience Study 

# Getting the top 10 countries with the most streamers
most_streamers_countries = youtubers_df['Country'].value_counts().head(10).index
most_streamers_countries


# Filtering the dataset for these countries and calculate the most popular categories for each
popular_categories_by_country = youtubers_df[youtubers_df['Country'].isin(most_streamers_countries)]
popular_categories_by_country

# Grouping by country and categories and count the occurrences
popular_categories_by_country = popular_categories_by_country.groupby(['Country', 'Categories']).size().reset_index(name='Counts')
popular_categories_by_country


# top category for each country
top_category_by_country = popular_categories_by_country.loc[popular_categories_by_country.groupby('Country')['Counts'].idxmax()]

top_category_by_country

category_by_country = youtubers_df.groupby(['Categories', 'Country']).agg({'Subscribers': 'mean', 'Visits': 'mean', 'Likes': 'mean', 'Comments': 'mean'}).reset_index()
category_by_country

top_categories2 = category_by_country.groupby('Categories')['Subscribers'].mean().nlargest(10).index

# the original dataframe to include only the top categories
filtered_data = category_by_country[category_by_country['Categories'].isin(top_categories2)]

# Create the plot
plt.figure(figsize=(14, 8))
sns.barplot(x='Subscribers', y='Categories', hue='Country', data=filtered_data, errorbar=None)
plt.title('Top Categories by Country Based on Average Subscribers')
plt.xlabel('Average Number of Subscribers')
plt.ylabel('Categories')
plt.legend(title='Country', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()

#  Perfomance Matrix

# the average number of subscribers, visits, likes, and comments
average_metrics = youtubers_df[['Subscribers', 'Visits', 'Likes', 'Comments']].mean()
average_metrics

import seaborn as sns

# Visualizing the average metrics
plt.figure(figsize=(10, 6))
sns.barplot(x=average_metrics.index, y=average_metrics.values)
plt.title('Average Number of Subscribers, Visits, Likes, and Comments')
plt.ylabel('Average Count')
plt.show()

# Content Categories

# the distribution of content categories and the number of streamers in each category
streamers_by_category = youtubers_df['Categories'].value_counts()
streamers_by_category.head(11)

# Visualizing the distribution of content categories and the number of streamers in each category
plt.figure(figsize=(10, 6))
sns.barplot(x=streamers_by_category.head(10).values, y=streamers_by_category.head(10).index)
plt.title('Number of Streamers by Content Category')
plt.xlabel('Number of Streamers')
plt.ylabel('Content Category')
plt.show()

# Benchmarking

# Identify streamers with above-average performance in terms of subscribers, visits, likes, and comments
above_average_streamers = youtubers_df[(youtubers_df['Subscribers'] > youtubers_df['Subscribers'].mean()) &
                                       (youtubers_df['Visits'] > youtubers_df['Visits'].mean()) &
                                       (youtubers_df['Likes'] > youtubers_df['Likes'].mean()) &
                                       (youtubers_df['Comments'] > youtubers_df['Comments'].mean())]
above_average_streamers.head(11)

# Content Recommendations

# content categories based on their content and audience engagement
streamers_by_category = youtubers_df.groupby('Categories').agg({'Subscribers': 'mean', 'Visits': 'mean', 'Likes': 'mean', 'Comments': 'mean'})
streamers_by_category

streamers_by_category.head()

top_streamers_by_category = youtubers_df.groupby('Categories').apply(lambda x: x.nlargest(1, 'Subscribers'))
top_streamers_by_category
