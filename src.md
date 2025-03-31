# Correlation analysys
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Load and prepare the shelter capacity data
shelter_data = pd.DataFrame({
    'Province': ['Newfoundland and Labrador', 'New Brunswick', 'Nova Scotia', 'Prince Edward Island', 
                 'Quebec', 'Ontario', 'Manitoba', 'Saskatchewan', 'Alberta', 'British Columbia', 
                 'Yukon', 'Northwest Territories', 'Nunavut'],
    'General_Beds': [4, 0, 0, 0, 114, 250, 507, 20, 1262, 545, 10, 0, 0],
    'Mens_Beds': [20, 35, 115, 7, 916, 1483, 0, 78, 304, 291, 0, 44, 22],
    'Womens_Beds': [0, 10, 36, 0, 259, 572, 0, 91, 6, 238, 0, 23, 12],
    'Youth_Beds': [17, 0, 20, 0, 197, 416, 34, 11, 45, 90, 4, 10, 0],
    'Family_Beds': [12, 0, 0, 0, 7, 496, 60, 0, 0, 30, 0, 32, 0],
    'Total_Beds': [53, 45, 171, 7, 1493, 3217, 601, 200, 1617, 1194, 14, 109, 34],
    'Total_Shelters': [5, 2, 5, 1, 45, 41, 9, 12, 11, 38, 2, 4, 2]
})

# Load and prepare the homelessness data
homelessness_data = pd.DataFrame({
    'Region': ['Atlantic', 'Quebec', 'Ontario', 'Prairies', 'British Columbia', 'Territories'],
    'Total_Homelessness': [7.3, 16.9, 38.0, 19.6, 17.7, 0.3]
})

# Map provinces to regions
region_mapping = {
    'Newfoundland and Labrador': 'Atlantic',
    'New Brunswick': 'Atlantic',
    'Nova Scotia': 'Atlantic',
    'Prince Edward Island': 'Atlantic',
    'Quebec': 'Quebec',
    'Ontario': 'Ontario',
    'Manitoba': 'Prairies',
    'Saskatchewan': 'Prairies',
    'Alberta': 'Prairies',
    'British Columbia': 'British Columbia',
    'Yukon': 'Territories',
    'Northwest Territories': 'Territories',
    'Nunavut': 'Territories'
}

shelter_data['Region'] = shelter_data['Province'].map(region_mapping)

# Aggregate shelter data by region
region_shelter_data = shelter_data.groupby('Region').sum().reset_index()

# Merge datasets
merged_data = pd.merge(region_shelter_data, homelessness_data, on='Region')

# Create normalized per capita metrics
merged_data['Beds_per_Homeless'] = merged_data['Total_Beds'] / merged_data['Total_Homelessness']
merged_data['Shelters_per_Homeless'] = merged_data['Total_Shelters'] / merged_data['Total_Homelessness']

# Select numerical columns for correlation
numerical_columns = ['General_Beds', 'Mens_Beds', 'Womens_Beds', 'Youth_Beds', 
                     'Family_Beds', 'Total_Beds', 'Total_Shelters', 'Total_Homelessness',
                     'Beds_per_Homeless', 'Shelters_per_Homeless']
correlation_data = merged_data[numerical_columns]

# Calculate correlation matrix
correlation_matrix = correlation_data.corr()

# Create heatmap
plt.figure(figsize=(14, 12))
sns.set(font_scale=1.2)

# Create heatmap with custom color map
heatmap = sns.heatmap(
    correlation_matrix, 
    annot=True, 
    cmap='coolwarm', 
    vmin=-1, 
    vmax=1, 
    center=0,
    square=True, 
    linewidths=.5,
    cbar_kws={"shrink": .8},
    fmt='.2f'
)

plt.title('Correlation Matrix: Shelter Capacity vs Homelessness Data', fontsize=18, pad=20)
plt.xticks(rotation=45, ha='right')
plt.tight_layout()

# Create a second visualization: scatter plot of total beds vs homelessness with regression line
plt.figure(figsize=(10, 8))
sns.set_style("whitegrid")
scatter = sns.regplot(
    x='Total_Homelessness', 
    y='Total_Beds', 
    data=merged_data, 
    scatter_kws={'s': 100, 'alpha': 0.7},
    line_kws={'color': 'red'}
)

# Add region labels
for i, row in merged_data.iterrows():
    plt.annotate(
        row['Region'], 
        (row['Total_Homelessness'], row['Total_Beds']), 
        xytext=(7, 0), 
        textcoords='offset points',
        fontsize=12,
        fontweight='bold'
    )

plt.title('Relationship between Total Homelessness and Total Shelter Beds by Region', fontsize=16)
plt.xlabel('Total Homelessness Rate (%)', fontsize=14)
plt.ylabel('Total Shelter Beds', fontsize=14)
plt.tight_layout()

# Create a third visualization: bar chart comparing beds per homeless person across regions
plt.figure(figsize=(12, 8))
merged_data = merged_data.sort_values('Beds_per_Homeless', ascending=False)
bars = sns.barplot(
    x='Region', 
    y='Beds_per_Homeless', 
    data=merged_data,
    palette='viridis'
)

plt.title('Shelter Beds per Homeless Person by Region', fontsize=16)
plt.xlabel('Region', fontsize=14)
plt.ylabel('Beds per Homeless Person', fontsize=14)
plt.xticks(rotation=45)

# Add value labels on top of bars
for i, v in enumerate(merged_data['Beds_per_Homeless']):
    plt.text(i, v + 1, f'{v:.1f}', ha='center', fontsize=12)

plt.tight_layout()

# Print correlation coefficients with homelessness
print("\nCorrelation with Total Homelessness:")
correlations_with_homelessness = correlation_matrix['Total_Homelessness'].sort_values(ascending=False)
print(correlations_with_homelessness)

# Print summary statistics
print("\nSummary Statistics by Region:")
summary_stats = merged_data[['Region', 'Total_Beds', 'Total_Homelessness', 'Beds_per_Homeless']]
print(summary_stats)

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import pearsonr

# Load shelter capacity data
shelter_data = pd.DataFrame({
    'Province': ['Newfoundland and Labrador', 'New Brunswick', 'Nova Scotia', 'Prince Edward Island', 
                 'Quebec', 'Ontario', 'Manitoba', 'Saskatchewan', 'Alberta', 'British Columbia', 
                 'Yukon', 'Northwest Territories', 'Nunavut'],
    'General_Beds': [4, 0, 0, 0, 114, 250, 507, 20, 1262, 545, 10, 0, 0],
    'Mens_Beds': [20, 35, 115, 7, 916, 1483, 0, 78, 304, 291, 0, 44, 22],
    'Womens_Beds': [0, 10, 36, 0, 259, 572, 0, 91, 6, 238, 0, 23, 12],
    'Youth_Beds': [17, 0, 20, 0, 197, 416, 34, 11, 45, 90, 4, 10, 0],
    'Family_Beds': [12, 0, 0, 0, 7, 496, 60, 0, 0, 30, 0, 32, 0],
    'Total_Beds': [53, 45, 171, 7, 1493, 3217, 601, 200, 1617, 1194, 14, 109, 34],
    'Total_Shelters': [5, 2, 5, 1, 45, 41, 9, 12, 11, 38, 2, 4, 2]
})

# Load homelessness data
homelessness_data = pd.DataFrame({
    'Region': ['Atlantic', 'Quebec', 'Ontario', 'Prairies', 'British Columbia', 'Territories'],
    'Total_Homelessness': [7.3, 16.9, 38.0, 19.6, 17.7, 0.3]
})

# Load demographic data - households
demographic_hh = pd.DataFrame({
    'Age_Group': ['15 to 34 years old', '35 to 44 years old', '45 to 54 years old', '55 to 64 years old', '65 years old and older'],
    'HH_Percent': [24, 22, 20, 19, 15]
})

# Load demographic data - general homeless
demographic_hml = pd.DataFrame({
    'Age_Group': ['15 to 34 years old', '35 to 44 years old', '45 to 54 years old', '55 to 64 years old', '65 years old and older'],
    'HML_Percent': [17.4, 22, 21, 27, 13.2]
})

# Gender data from homeless population
gender_data = pd.DataFrame({
    'Gender': ['Males', 'Females'],
    'Percent': [54, 46]
})

# Map provinces to regions
region_mapping = {
    'Newfoundland and Labrador': 'Atlantic',
    'New Brunswick': 'Atlantic',
    'Nova Scotia': 'Atlantic',
    'Prince Edward Island': 'Atlantic',
    'Quebec': 'Quebec',
    'Ontario': 'Ontario',
    'Manitoba': 'Prairies',
    'Saskatchewan': 'Prairies',
    'Alberta': 'Prairies',
    'British Columbia': 'British Columbia',
    'Yukon': 'Territories',
    'Northwest Territories': 'Territories',
    'Nunavut': 'Territories'
}

shelter_data['Region'] = shelter_data['Province'].map(region_mapping)

# Aggregate shelter data by region
region_shelter_data = shelter_data.groupby('Region').sum().reset_index()

# Merge shelter and homelessness data
merged_data = pd.merge(region_shelter_data, homelessness_data, on='Region')

# Merge demographic data
demographic_data = pd.merge(demographic_hh, demographic_hml, on='Age_Group')

# Calculate correlations between different datasets
# First, calculate shelter capacity vs homelessness correlations
shelter_homeless_corr = merged_data.drop(['Region', 'Province'], axis=1, errors='ignore').corr()

# Calculate correlation between HH and HML demographic percentages
demo_corr = demographic_data[['HH_Percent', 'HML_Percent']].corr().iloc[0, 1]

# Create a figure with multiple subplots
fig = plt.figure(figsize=(20, 15))

# 1. Shelter capacity vs homelessness correlations
ax1 = plt.subplot(2, 2, 1)
sns.heatmap(shelter_homeless_corr, annot=True, cmap='coolwarm', vmin=-1, vmax=1, center=0, ax=ax1)
ax1.set_title('Shelter Capacity vs Homelessness Correlation Matrix', fontsize=16)

# 2. Age demographic comparison
ax2 = plt.subplot(2, 2, 2)
width = 0.35
x = np.arange(len(demographic_data['Age_Group']))
ax2.bar(x - width/2, demographic_data['HH_Percent'], width, label='Household Demographics')
ax2.bar(x + width/2, demographic_data['HML_Percent'], width, label='Homeless Demographics')
ax2.set_xticks(x)
ax2.set_xticklabels(demographic_data['Age_Group'], rotation=45, ha='right')
ax2.set_title('Age Demographics: Household vs Homeless Population', fontsize=16)
ax2.set_ylabel('Percent')
ax2.legend()

# Add correlation coefficient
corr_text = f'Correlation: {demo_corr:.2f}'
ax2.text(0.5, 0.95, corr_text, transform=ax2.transAxes, ha='center', fontsize=12, bbox=dict(facecolor='white', alpha=0.5))

# 3. Gender split in homeless population
ax3 = plt.subplot(2, 2, 3)
ax3.pie(gender_data['Percent'], labels=gender_data['Gender'], autopct='%1.1f%%', startangle=90)
ax3.set_title('Gender Distribution in Homeless Population', fontsize=16)

# 4. Beds per homeless person by region
ax4 = plt.subplot(2, 2, 4)
merged_data['Beds_per_Homeless'] = merged_data['Total_Beds'] / merged_data['Total_Homelessness']
merged_data_sorted = merged_data.sort_values('Beds_per_Homeless', ascending=False)
sns.barplot(x='Region', y='Beds_per_Homeless', data=merged_data_sorted, ax=ax4)
ax4.set_title('Shelter Beds per Homeless Person by Region', fontsize=16)
ax4.set_xticklabels(ax4.get_xticklabels(), rotation=45, ha='right')
ax4.set_ylabel('Beds per Homeless Person')

# Add text labels to the bars
for i, v in enumerate(merged_data_sorted['Beds_per_Homeless']):
    ax4.text(i, v + 0.5, f'{v:.1f}', ha='center', fontsize=10)

plt.tight_layout()

# Additional analysis: Correlation between age groups and shelter types
# Create a correlation matrix to analyze relationships between datasets
# First, create a combined dataset with all relevant metrics

# Calculate demographic age group percentages for each shelter type
shelter_types = ['General_Beds', 'Mens_Beds', 'Womens_Beds', 'Youth_Beds', 'Family_Beds']
age_groups = demographic_data['Age_Group'].tolist()

# Create a dataframe to store the calculated correlations
correlation_df = pd.DataFrame(columns=['Variable_1', 'Variable_2', 'Correlation'])

# Calculate correlations between demographic percentages and shelter beds
for shelter_type in shelter_types:
    for age_col in ['HH_Percent', 'HML_Percent']:
        # Since we don't have direct correlations between these datasets, we'll note this limitation
        correlation_df = correlation_df.append({
            'Variable_1': shelter_type,
            'Variable_2': age_col,
            'Correlation': np.nan  # Cannot calculate direct correlation without regional demographic data
        }, ignore_index=True)

# Calculate correlations between shelter types
for i, shelter_type1 in enumerate(shelter_types):
    for shelter_type2 in shelter_types[i:]:
        if shelter_type1 != shelter_type2:
            corr = merged_data[shelter_type1].corr(merged_data[shelter_type2])
            correlation_df = correlation_df.append({
                'Variable_1': shelter_type1,
                'Variable_2': shelter_type2,
                'Correlation': corr
            }, ignore_index=True)

# Calculate correlations with homelessness
for shelter_type in shelter_types:
    corr = merged_data[shelter_type].corr(merged_data['Total_Homelessness'])
    correlation_df = correlation_df.append({
        'Variable_1': shelter_type,
        'Variable_2': 'Total_Homelessness',
        'Correlation': corr
    }, ignore_index=True)

# Create integrated correlation matrix
# Combine all numeric variables that can be directly correlated
plt.figure(figsize=(14, 12))
corr_matrix = merged_data.drop(['Region', 'Province'], axis=1, errors='ignore').corr()
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', vmin=-1, vmax=1, center=0)
plt.title('Integrated Correlation Matrix', fontsize=18)
plt.tight_layout()

plt.show()

# Print key findings
print("Key Findings from Correlation Analysis:")
print(f"1. Correlation between household and homeless age demographics: {demo_corr:.2f}")
print("2. Correlations between shelter types and homelessness:")
for _, row in correlation_df[correlation_df['Variable_2'] == 'Total_Homelessness'].iterrows():
    print(f"   - {row['Variable_1']} correlation with Total_Homelessness: {row['Correlation']:.2f}")
print("3. Gender distribution in homeless population:")
for _, row in gender_data.iterrows():
    print(f"   - {row['Gender']}: {row['Percent']}%")
print("4. Age demographics comparison:")
for _, row in demographic_data.iterrows():
print(f"   - {row['Age_Group']}: Household {row['HH_Percent']}%, Homeless {row['HML_Percent']}%")
