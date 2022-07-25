# KPrototypeClustering

# Library
    import pandas as pd
    import numpy as np
    from plotnine import *
    import plotnine
    from kmodes.kprototypes import KPrototypes
    import warnings
    warnings.filterwarnings('ignore', category = FutureWarning)

# Format scientific notation from Pandas
    pd.set_option('display.float_format', lambda x: '%.3f' % x)

# Import data
    data = pd.read_excel("C:/Galuh/clustering program/Clustering Juni.xlsx", sheet_name='tempo.institute') #CHANGE PATH
    data

# Select Kolom
    selected_data = data.iloc[:, 7:13] #makesure you select fit column
    selected_data.info()


    catColumnsPos = [selected_data.columns.get_loc(col) for col in list(selected_data.select_dtypes('object').columns)]
    print('Categorical columns           : {}'.format(list(selected_data.select_dtypes('object').columns)))
    print('Categorical columns position  : {}'.format(catColumnsPos))

# Fit the cluster
    kprototype = KPrototypes(n_jobs = -1, n_clusters = 3, init = 'Huang', random_state = 0)
    kprototype.fit_predict(selected_data, categorical = catColumnsPos)

# Add the cluster to the dataframe
    selected_data['Cluster Labels'] = kprototype.labels_
    selected_data['Segment'] = selected_data['Cluster Labels'].map({0:'First', 1:'Second', 2:'Third'})
    
# Order the cluster
    selected_data['Segment'] = selected_data['Segment'].astype('category')
    selected_data['Segment'] = selected_data['Segment'].cat.reorder_categories(['First','Second','Third'])
    selected_data.insert(0, "Cluster", selected_data['Segment'], True)
    selected_data

# Save All Cluster Table
    selected_data.to_csv('C:/Galuh/clustering program/tempo.institute_juni.csv', index=False) #CHANGE PATH

# Cluster interpretation
    selected_data.rename(columns = {'Cluster Labels':'Total'}, inplace = True)
    selected_data.groupby('Segment').agg(
     {
         'category': lambda x: x.value_counts().index[0],
         'Day': lambda x: x.value_counts().index[0],
         'Range': lambda x: x.value_counts().index[0],
         'video_play_count': 'mean',
         'like': 'mean',
         'comment': 'mean'
          }
    ).reset_index()
