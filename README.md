<h2 align=center>Analyze Worldwide Box Office Revenue with Plotly and Python</h2>
<img src="revenue.png">


### (Part 1) Libraries


```python
import numpy as np
import pandas as pd
pd.set_option('max_columns', None)
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
plt.style.use('ggplot')
import datetime
import lightgbm as lgb
from scipy import stats
from scipy.sparse import hstack, csr_matrix
from sklearn.model_selection import train_test_split, KFold
from wordcloud import WordCloud
from collections import Counter
from nltk.corpus import stopwords
from nltk.util import ngrams
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer
from sklearn.preprocessing import StandardScaler
import nltk
nltk.download('stopwords')
stop = set(stopwords.words('english'))
import os
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
import plotly.tools as tls
import xgboost as xgb
import lightgbm as lgb
from sklearn import model_selection
from sklearn.metrics import accuracy_score
import json
import ast
from urllib.request import urlopen
from PIL import Image
from sklearn.preprocessing import LabelEncoder
import time
from sklearn.metrics import mean_squared_error
from sklearn.linear_model import LinearRegression
from sklearn import linear_model
```

    [nltk_data] Downloading package stopwords to /home/rhyme/nltk_data...
    [nltk_data]   Unzipping corpora/stopwords.zip.
    


<script type="text/javascript">
window.PlotlyConfig = {MathJaxConfig: 'local'};
if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
if (typeof require !== 'undefined') {
require.undef("plotly");
requirejs.config({
    paths: {
        'plotly': ['https://cdn.plot.ly/plotly-latest.min']
    }
});
require(['plotly'], function(Plotly) {
    window._Plotly = Plotly;
});
}
</script>



### (Part 1) Data Loading and Exploration


```python
train = pd.read_csv('data/train.csv')
test = pd.read_csv('data/test.csv')
```


```python
train.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>budget</th>
      <th>homepage</th>
      <th>imdb_id</th>
      <th>original_language</th>
      <th>original_title</th>
      <th>overview</th>
      <th>popularity</th>
      <th>poster_path</th>
      <th>release_date</th>
      <th>runtime</th>
      <th>status</th>
      <th>tagline</th>
      <th>title</th>
      <th>Keywords</th>
      <th>revenue</th>
      <th>collection_name</th>
      <th>has_collection</th>
      <th>num_genres</th>
      <th>all_genres</th>
      <th>genre_Drama</th>
      <th>genre_Comedy</th>
      <th>genre_Thriller</th>
      <th>genre_Action</th>
      <th>genre_Romance</th>
      <th>genre_Crime</th>
      <th>genre_Adventure</th>
      <th>genre_Horror</th>
      <th>genre_Science Fiction</th>
      <th>genre_Family</th>
      <th>genre_Fantasy</th>
      <th>genre_Mystery</th>
      <th>genre_Animation</th>
      <th>genre_History</th>
      <th>genre_Music</th>
      <th>num_companies</th>
      <th>production_company_Warner Bros.</th>
      <th>production_company_Universal Pictures</th>
      <th>production_company_Paramount Pictures</th>
      <th>production_company_Twentieth Century Fox Film Corporation</th>
      <th>production_company_Columbia Pictures</th>
      <th>production_company_Metro-Goldwyn-Mayer (MGM)</th>
      <th>production_company_New Line Cinema</th>
      <th>production_company_Touchstone Pictures</th>
      <th>production_company_Walt Disney Pictures</th>
      <th>production_company_Columbia Pictures Corporation</th>
      <th>production_company_TriStar Pictures</th>
      <th>production_company_Relativity Media</th>
      <th>production_company_Canal+</th>
      <th>production_company_United Artists</th>
      <th>production_company_Miramax Films</th>
      <th>production_company_Village Roadshow Pictures</th>
      <th>production_company_Regency Enterprises</th>
      <th>production_company_BBC Films</th>
      <th>production_company_Dune Entertainment</th>
      <th>production_company_Working Title Films</th>
      <th>production_company_Fox Searchlight Pictures</th>
      <th>production_company_StudioCanal</th>
      <th>production_company_Lionsgate</th>
      <th>production_company_DreamWorks SKG</th>
      <th>production_company_Fox 2000 Pictures</th>
      <th>production_company_Summit Entertainment</th>
      <th>production_company_Hollywood Pictures</th>
      <th>production_company_Orion Pictures</th>
      <th>production_company_Amblin Entertainment</th>
      <th>production_company_Dimension Films</th>
      <th>num_countries</th>
      <th>production_country_United States of America</th>
      <th>production_country_United Kingdom</th>
      <th>production_country_France</th>
      <th>production_country_Germany</th>
      <th>production_country_Canada</th>
      <th>production_country_India</th>
      <th>production_country_Italy</th>
      <th>production_country_Japan</th>
      <th>production_country_Australia</th>
      <th>production_country_Russia</th>
      <th>production_country_Spain</th>
      <th>production_country_China</th>
      <th>production_country_Hong Kong</th>
      <th>production_country_Ireland</th>
      <th>production_country_Belgium</th>
      <th>production_country_South Korea</th>
      <th>production_country_Mexico</th>
      <th>production_country_Sweden</th>
      <th>production_country_New Zealand</th>
      <th>production_country_Netherlands</th>
      <th>production_country_Czech Republic</th>
      <th>production_country_Denmark</th>
      <th>production_country_Brazil</th>
      <th>production_country_Luxembourg</th>
      <th>production_country_South Africa</th>
      <th>num_languages</th>
      <th>language_English</th>
      <th>language_Français</th>
      <th>language_Español</th>
      <th>language_Deutsch</th>
      <th>language_Pусский</th>
      <th>language_Italiano</th>
      <th>language_日本語</th>
      <th>language_普通话</th>
      <th>language_हिन्दी</th>
      <th>language_</th>
      <th>language_Português</th>
      <th>language_العربية</th>
      <th>language_한국어/조선말</th>
      <th>language_广州话 / 廣州話</th>
      <th>language_தமிழ்</th>
      <th>language_Polski</th>
      <th>language_Magyar</th>
      <th>language_Latin</th>
      <th>language_svenska</th>
      <th>language_ภาษาไทย</th>
      <th>language_Český</th>
      <th>language_עִבְרִית</th>
      <th>language_ελληνικά</th>
      <th>language_Türkçe</th>
      <th>language_Dansk</th>
      <th>language_Nederlands</th>
      <th>language_فارسی</th>
      <th>language_Tiếng Việt</th>
      <th>language_اردو</th>
      <th>language_Română</th>
      <th>num_cast</th>
      <th>cast_name_Samuel L. Jackson</th>
      <th>cast_name_Robert De Niro</th>
      <th>cast_name_Morgan Freeman</th>
      <th>cast_name_J.K. Simmons</th>
      <th>cast_name_Bruce Willis</th>
      <th>cast_name_Liam Neeson</th>
      <th>cast_name_Susan Sarandon</th>
      <th>cast_name_Bruce McGill</th>
      <th>cast_name_John Turturro</th>
      <th>cast_name_Forest Whitaker</th>
      <th>cast_name_Willem Dafoe</th>
      <th>cast_name_Bill Murray</th>
      <th>cast_name_Owen Wilson</th>
      <th>cast_name_Nicolas Cage</th>
      <th>cast_name_Sylvester Stallone</th>
      <th>genders_0_cast</th>
      <th>genders_1_cast</th>
      <th>genders_2_cast</th>
      <th>cast_character_</th>
      <th>cast_character_Himself</th>
      <th>cast_character_Herself</th>
      <th>cast_character_Dancer</th>
      <th>cast_character_Additional Voices (voice)</th>
      <th>cast_character_Doctor</th>
      <th>cast_character_Reporter</th>
      <th>cast_character_Waitress</th>
      <th>cast_character_Nurse</th>
      <th>cast_character_Bartender</th>
      <th>cast_character_Jack</th>
      <th>cast_character_Debutante</th>
      <th>cast_character_Security Guard</th>
      <th>cast_character_Paul</th>
      <th>cast_character_Frank</th>
      <th>num_crew</th>
      <th>crew_name_Avy Kaufman</th>
      <th>crew_name_Robert Rodriguez</th>
      <th>crew_name_Deborah Aquila</th>
      <th>crew_name_James Newton Howard</th>
      <th>crew_name_Mary Vernieu</th>
      <th>crew_name_Steven Spielberg</th>
      <th>crew_name_Luc Besson</th>
      <th>crew_name_Jerry Goldsmith</th>
      <th>crew_name_Francine Maisler</th>
      <th>crew_name_Tricia Wood</th>
      <th>crew_name_James Horner</th>
      <th>crew_name_Kerry Barden</th>
      <th>crew_name_Bob Weinstein</th>
      <th>crew_name_Harvey Weinstein</th>
      <th>crew_name_Janet Hirshenson</th>
      <th>genders_0_crew</th>
      <th>genders_1_crew</th>
      <th>genders_2_crew</th>
      <th>jobs_Producer</th>
      <th>jobs_Executive Producer</th>
      <th>jobs_Director</th>
      <th>jobs_Screenplay</th>
      <th>jobs_Editor</th>
      <th>jobs_Casting</th>
      <th>jobs_Director of Photography</th>
      <th>jobs_Original Music Composer</th>
      <th>jobs_Art Direction</th>
      <th>jobs_Production Design</th>
      <th>jobs_Costume Design</th>
      <th>jobs_Writer</th>
      <th>jobs_Set Decoration</th>
      <th>jobs_Makeup Artist</th>
      <th>jobs_Sound Re-Recording Mixer</th>
      <th>departments_Production</th>
      <th>departments_Sound</th>
      <th>departments_Art</th>
      <th>departments_Crew</th>
      <th>departments_Writing</th>
      <th>departments_Costume &amp; Make-Up</th>
      <th>departments_Camera</th>
      <th>departments_Directing</th>
      <th>departments_Editing</th>
      <th>departments_Visual Effects</th>
      <th>departments_Lighting</th>
      <th>departments_Actors</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>14000000</td>
      <td>NaN</td>
      <td>tt2637294</td>
      <td>en</td>
      <td>Hot Tub Time Machine 2</td>
      <td>When Lou, who has become the "father of the In...</td>
      <td>6.575393</td>
      <td>/tQtWuwvMf0hCc2QR2tkolwl7c3c.jpg</td>
      <td>2/20/15</td>
      <td>93.0</td>
      <td>Released</td>
      <td>The Laws of Space and Time are About to be Vio...</td>
      <td>Hot Tub Time Machine 2</td>
      <td>[{'id': 4379, 'name': 'time travel'}, {'id': 9...</td>
      <td>12314651</td>
      <td>Hot Tub Time Machine Collection</td>
      <td>1</td>
      <td>1</td>
      <td>Comedy</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>24</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>6</td>
      <td>8</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>72</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>59</td>
      <td>0</td>
      <td>13</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>2</td>
      <td>9</td>
      <td>10</td>
      <td>12</td>
      <td>4</td>
      <td>2</td>
      <td>13</td>
      <td>8</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>40000000</td>
      <td>NaN</td>
      <td>tt0368933</td>
      <td>en</td>
      <td>The Princess Diaries 2: Royal Engagement</td>
      <td>Mia Thermopolis is now a college graduate and ...</td>
      <td>8.248895</td>
      <td>/w9Z7A0GHEhIp7etpj0vyKOeU1Wx.jpg</td>
      <td>8/6/04</td>
      <td>113.0</td>
      <td>Released</td>
      <td>It can take a lifetime to find true love; she'...</td>
      <td>The Princess Diaries 2: Royal Engagement</td>
      <td>[{'id': 2505, 'name': 'coronation'}, {'id': 42...</td>
      <td>95149435</td>
      <td>The Princess Diaries Collection</td>
      <td>1</td>
      <td>4</td>
      <td>Comedy Drama Family Romance</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>20</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>10</td>
      <td>10</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>3300000</td>
      <td>http://sonyclassics.com/whiplash/</td>
      <td>tt2582802</td>
      <td>en</td>
      <td>Whiplash</td>
      <td>Under the direction of a ruthless instructor, ...</td>
      <td>64.299990</td>
      <td>/lIv1QinFqz4dlp5U4lQ6HaiskOZ.jpg</td>
      <td>10/10/14</td>
      <td>105.0</td>
      <td>Released</td>
      <td>The road to greatness can take you to the edge.</td>
      <td>Whiplash</td>
      <td>[{'id': 1416, 'name': 'jazz'}, {'id': 1523, 'n...</td>
      <td>13092000</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Drama</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>51</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>31</td>
      <td>7</td>
      <td>13</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>64</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>49</td>
      <td>4</td>
      <td>11</td>
      <td>4</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>18</td>
      <td>9</td>
      <td>5</td>
      <td>9</td>
      <td>1</td>
      <td>5</td>
      <td>4</td>
      <td>3</td>
      <td>6</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>1200000</td>
      <td>http://kahaanithefilm.com/</td>
      <td>tt1821480</td>
      <td>hi</td>
      <td>Kahaani</td>
      <td>Vidya Bagchi (Vidya Balan) arrives in Kolkata ...</td>
      <td>3.174936</td>
      <td>/aTXRaPrWSinhcmCrcfJK17urp3F.jpg</td>
      <td>3/9/12</td>
      <td>122.0</td>
      <td>Released</td>
      <td>NaN</td>
      <td>Kahaani</td>
      <td>[{'id': 10092, 'name': 'mystery'}, {'id': 1054...</td>
      <td>16000000</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>Drama Thriller</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>0</td>
      <td>NaN</td>
      <td>tt1380152</td>
      <td>ko</td>
      <td>마린보이</td>
      <td>Marine Boy is the story of a former national s...</td>
      <td>1.148070</td>
      <td>/m22s7zvkVFDU9ir56PiiqIEWFdT.jpg</td>
      <td>2/5/09</td>
      <td>118.0</td>
      <td>Released</td>
      <td>NaN</td>
      <td>Marine Boy</td>
      <td>{}</td>
      <td>3923970</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>Action Thriller</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



 

 

### (Part 1) Visualizing the Target Distribution


```python
fig, ax = plt.subplots(figsize = (16, 6))
plt.subplot(1, 2, 1)
plt.hist(train['revenue']);
plt.title('Distribution of revenue');
plt.subplot(1, 2, 2)
plt.hist(np.log1p(train['revenue']));
plt.title('Distribution of log of revenue');
```


    
![png](output_10_0.png)
    



```python
train['log_revenue'] = np.log1p(train['revenue'])
```

  

### (Part 1) Relationship between Film Revenue and Budget


```python
fig, ax = plt.subplots(figsize = (16, 6))
plt.subplot(1, 2, 1)
plt.hist(train['budget']);
plt.title('Distribution of budget');
plt.subplot(1, 2, 2)
plt.hist(np.log1p(train['budget']));
plt.title('Distribution of log of budget');
```


    
![png](output_14_0.png)
    



```python
plt.figure(figsize=(16, 8))
plt.subplot(1, 2, 1)
plt.scatter(train['budget'], train['revenue'])
plt.title('Revenue vs budget');
plt.subplot(1, 2, 2)
plt.scatter(np.log1p(train['budget']), train['log_revenue'])
plt.title('Log Revenue vs log budget');
```


    
![png](output_15_0.png)
    



```python
train['log_budget'] = np.log1p(train['budget'])
test['log_budget'] = np.log1p(test['budget'])
```

 

### (Part 1) Does having an Official Homepage Affect Revenue?


```python
train['homepage'].value_counts().head(10)
```




    http://www.transformersmovie.com/                 4
    http://www.lordoftherings.net/                    2
    http://www.thehobbit.com/                         2
    http://www.jackgoesboatingmovie.com/              1
    http://www.sonypictures.com/movies/theholiday/    1
    http://3bogatirya.ru/                             1
    http://licensetowedthemovie.warnerbros.com/       1
    http://eleanorrigby-movie.com/                    1
    http://www.fastandfuriousmovie.net                1
    http://www.splitmovie.com/                        1
    Name: homepage, dtype: int64




```python
train['has_homepage'] = 0
train.loc[train['homepage'].isnull() == False, 'has_homepage'] = 1
test['has_homepage'] = 0
test.loc[test['homepage'].isnull() == False, 'has_homepage'] = 1
```


```python
sns.catplot(x='has_homepage', y='revenue', data=train);
plt.title('Revenue for film with and without homepage');
```


    
![png](output_21_0.png)
    


 

### (Part 1) Distribution of Languages in Film


```python
plt.figure(figsize=(16, 8))
plt.subplot(1, 2, 1)
sns.boxplot(x='original_language', y='revenue', data=train.loc[train['original_language'].isin(train['original_language'].value_counts().head(10).index)]);
plt.title('Mean revenue per language');
plt.subplot(1, 2, 2)
sns.boxplot(x='original_language', y='log_revenue', data=train.loc[train['original_language'].isin(train['original_language'].value_counts().head(10).index)]);
plt.title('Mean log revenue per language');
```


    
![png](output_24_0.png)
    


 

### (Part 1) Frequent Words in Film Titles and Discriptions


```python
plt.figure(figsize = (12, 12))
text = ' '.join(train['original_title'].values)
wordcloud = WordCloud(max_font_size=None, background_color='white', width=1200, height=1000).generate(text)
plt.imshow(wordcloud)
plt.title('Top words in titles')
plt.axis("off")
plt.show()
```


    
![png](output_27_0.png)
    



```python
plt.figure(figsize = (12, 12))
text = ' '.join(train['overview'].fillna('').values)
wordcloud = WordCloud(max_font_size=None, background_color='white', width=1200, height=1000).generate(text)
plt.imshow(wordcloud)
plt.title('Top words in overview')
plt.axis("off")
plt.show()
```


    
![png](output_28_0.png)
    


### (Part 1) Do Film Descriptions Impact Revenue?


```python
import eli5

vectorizer = TfidfVectorizer(
            sublinear_tf=True,
            analyzer='word',
            token_pattern=r'\w{1,}',
            ngram_range=(1, 2),
            min_df=5)

overview_text = vectorizer.fit_transform(train['overview'].fillna(''))
linreg = LinearRegression()
linreg.fit(overview_text, train['log_revenue'])
eli5.show_weights(linreg, vec=vectorizer, top=20, feature_filter=lambda x: x != '<BIAS>')
```





    <style>
    table.eli5-weights tr:hover {
        filter: brightness(85%);
    }
</style>
































        <p style="margin-bottom: 0.5em; margin-top: 0em">
            <b>

        y

</b>

top features
        </p>

    <table class="eli5-weights"
           style="border-collapse: collapse; border: none; margin-top: 0em; table-layout: auto; margin-bottom: 2em;">
        <thead>
        <tr style="border: none;">

                <th style="padding: 0 1em 0 0.5em; text-align: right; border: none;" title="Feature weights. Note that weights do not account for feature value scales, so if feature values have different scales, features with highest weights might not be the most important.">
                    Weight<sup>?</sup>
                </th>

            <th style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">Feature</th>

        </tr>
        </thead>
        <tbody>

            <tr style="background-color: hsl(120, 100.00%, 85.12%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        +10.086
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        bombing
    </td>

</tr>

            <tr style="background-color: hsl(120, 100.00%, 85.99%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        +9.263
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        complications
    </td>

</tr>


            <tr style="background-color: hsl(120, 100.00%, 85.99%); border: none;">
                <td colspan="2" style="padding: 0 0.5em 0 0.5em; text-align: center; border: none; white-space: nowrap;">
                    <i>&hellip; 3742 more positive &hellip;</i>
                </td>
            </tr>



            <tr style="background-color: hsl(0, 100.00%, 86.08%); border: none;">
                <td colspan="2" style="padding: 0 0.5em 0 0.5em; text-align: center; border: none; white-space: nowrap;">
                    <i>&hellip; 3431 more negative &hellip;</i>
                </td>
            </tr>


            <tr style="background-color: hsl(0, 100.00%, 86.08%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.170
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        critic
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 86.06%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.195
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        status
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 85.92%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.329
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        18
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 85.89%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.356
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        politicians
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 85.78%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.456
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        life they
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 85.47%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.752
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        violence
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 85.36%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.860
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        she will
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 85.35%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -9.864
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        who also
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 85.01%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -10.199
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        casino
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 84.94%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -10.261
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        escape and
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 84.77%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -10.427
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        receiving
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 84.55%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -10.646
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        kept
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 84.54%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -10.653
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        attracted to
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 84.36%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -10.835
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        sally
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 83.41%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -11.791
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        and be
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 82.92%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -12.290
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        campaign
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 81.46%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -13.815
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        mike
    </td>

</tr>

            <tr style="background-color: hsl(0, 100.00%, 80.00%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        -15.395
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        woman from
    </td>

</tr>


        </tbody>
    </table>

















































```python
print('Target value:', train['log_revenue'][1000])
eli5.show_prediction(linreg, doc=train['overview'].values[1000], vec=vectorizer)
```

    Target value: 16.44583954907521
    





    <style>
    table.eli5-weights tr:hover {
        filter: brightness(85%);
    }
</style>


































        <p style="margin-bottom: 0.5em; margin-top: 0em">
            <b>

        y

</b>


    (score <b>16.446</b>)

top features
        </p>

    <table class="eli5-weights"
           style="border-collapse: collapse; border: none; margin-top: 0em; table-layout: auto; margin-bottom: 2em;">
        <thead>
        <tr style="border: none;">

                <th style="padding: 0 1em 0 0.5em; text-align: right; border: none;" title="Feature contribution already accounts for the feature value (for linear models, contribution = weight * feature value), and the sum of feature contributions is equal to the score or, for some classifiers, to the probability. Feature values are shown if &quot;show_feature_values&quot; is True.">
                    Contribution<sup>?</sup>
                </th>

            <th style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">Feature</th>

        </tr>
        </thead>
        <tbody>

            <tr style="background-color: hsl(120, 100.00%, 80.00%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        +15.962
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        &lt;BIAS&gt;
    </td>

</tr>

            <tr style="background-color: hsl(120, 100.00%, 98.27%); border: none;">
    <td style="padding: 0 1em 0 0.5em; text-align: right; border: none;">
        +0.484
    </td>
    <td style="padding: 0 0.5em 0 0.5em; text-align: left; border: none;">
        Highlighted in text (sum)
    </td>

</tr>






        </tbody>
    </table>





    <p style="margin-bottom: 2.5em; margin-top:-0.5em;">
        <span style="background-color: hsl(120, 100.00%, 99.46%); opacity: 0.80" title="0.002">when</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 98.97%); opacity: 0.80" title="-0.006">elizabeth</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 82.63%); opacity: 0.86" title="-0.359">returns</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 86.14%); opacity: 0.84" title="0.260">to</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 79.97%); opacity: 0.87" title="0.440">her</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 99.65%); opacity: 0.80" title="-0.001">mother</span><span style="opacity: 0.80">&#x27;</span><span style="background-color: hsl(120, 100.00%, 68.43%); opacity: 0.94" title="0.843">s</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 60.00%); opacity: 1.00" title="1.182">home</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 95.98%); opacity: 0.81" title="-0.044">after</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 91.06%); opacity: 0.82" title="-0.139">her</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 93.31%); opacity: 0.82" title="0.092">marriage</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 92.02%); opacity: 0.82" title="0.118">breaks</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 94.91%); opacity: 0.81" title="0.062">up</span><span style="opacity: 0.80">, </span><span style="background-color: hsl(0, 100.00%, 93.32%); opacity: 0.82" title="-0.092">she</span><span style="opacity: 0.80"> recreates </span><span style="background-color: hsl(120, 100.00%, 99.78%); opacity: 0.80" title="0.001">her</span><span style="opacity: 0.80"> imaginary </span><span style="background-color: hsl(120, 100.00%, 86.01%); opacity: 0.84" title="0.264">childhood</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 85.29%); opacity: 0.85" title="-0.283">friend</span><span style="opacity: 0.80">, </span><span style="background-color: hsl(120, 100.00%, 80.31%); opacity: 0.87" title="0.429">fred</span><span style="opacity: 0.80">, </span><span style="background-color: hsl(0, 100.00%, 94.40%); opacity: 0.81" title="-0.071">to</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 86.57%); opacity: 0.84" title="-0.249">escape</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 95.87%); opacity: 0.81" title="0.046">from</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 83.95%); opacity: 0.85" title="-0.321">the</span><span style="opacity: 0.80"> trauma </span><span style="background-color: hsl(0, 100.00%, 94.93%); opacity: 0.81" title="-0.062">of</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 82.51%); opacity: 0.86" title="-0.363">losing</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 88.87%); opacity: 0.83" title="-0.190">her</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 88.65%); opacity: 0.83" title="0.195">husband</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 93.77%); opacity: 0.81" title="0.083">and</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 79.62%); opacity: 0.88" title="0.451">her</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 85.25%); opacity: 0.85" title="-0.284">job</span><span style="opacity: 0.80">. </span><span style="background-color: hsl(0, 100.00%, 73.49%); opacity: 0.91" title="-0.657">in</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 91.65%); opacity: 0.82" title="-0.126">between</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 60.30%); opacity: 1.00" title="1.169">the</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 63.96%); opacity: 0.97" title="1.019">chaos</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 74.46%); opacity: 0.91" title="0.623">and</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 68.65%); opacity: 0.94" title="-0.835">mayhem</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 95.02%); opacity: 0.81" title="0.060">that</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 80.31%); opacity: 0.87" title="0.429">fred</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 81.58%); opacity: 0.87" title="0.390">creates</span><span style="opacity: 0.80">, </span><span style="background-color: hsl(0, 100.00%, 98.97%); opacity: 0.80" title="-0.006">elizabeth</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 81.46%); opacity: 0.87" title="-0.394">attempts</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 94.87%); opacity: 0.81" title="0.063">to</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 90.20%); opacity: 0.83" title="0.158">win</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 79.57%); opacity: 0.88" title="-0.453">back</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 95.67%); opacity: 0.81" title="-0.049">her</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 88.65%); opacity: 0.83" title="0.195">husband</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 67.10%); opacity: 0.95" title="-0.894">and</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(0, 100.00%, 67.66%); opacity: 0.95" title="-0.872">return</span><span style="opacity: 0.80"> </span><span style="background-color: hsl(120, 100.00%, 74.06%); opacity: 0.91" title="0.637">to</span><span style="opacity: 0.80"> normality.</span>
    </p>










































 

### Task 2: Analyzing Movie Release Dates
***
Note: If you are starting the notebook from this task, you can run cells from all the previous tasks in the kernel by going to the top menu and Kernel > Restart and Run All
***


```python
test.loc[test['release_date'].isnull() == False, 'release_date'].head()
```




    0    7/14/07
    1    5/19/58
    2    5/23/97
    3     9/4/10
    4    2/11/05
    Name: release_date, dtype: object




```python

```

 

### Task 3: Preprocessing Features


```python
def fix_date(x):
    year = x.split('/')[2]
    if int(year) <= 19:
        return x[:-2] + '20' + year
    else:
        return x[:-2] + '19' + year
```


```python
test.loc[test['release_date'].isnull() == True].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>budget</th>
      <th>homepage</th>
      <th>imdb_id</th>
      <th>original_language</th>
      <th>original_title</th>
      <th>overview</th>
      <th>popularity</th>
      <th>poster_path</th>
      <th>release_date</th>
      <th>runtime</th>
      <th>status</th>
      <th>tagline</th>
      <th>title</th>
      <th>Keywords</th>
      <th>collection_name</th>
      <th>has_collection</th>
      <th>num_genres</th>
      <th>all_genres</th>
      <th>genre_Drama</th>
      <th>genre_Comedy</th>
      <th>genre_Thriller</th>
      <th>genre_Action</th>
      <th>genre_Romance</th>
      <th>genre_Crime</th>
      <th>genre_Adventure</th>
      <th>genre_Horror</th>
      <th>genre_Science Fiction</th>
      <th>genre_Family</th>
      <th>genre_Fantasy</th>
      <th>genre_Mystery</th>
      <th>genre_Animation</th>
      <th>genre_History</th>
      <th>genre_Music</th>
      <th>num_companies</th>
      <th>production_company_Warner Bros.</th>
      <th>production_company_Universal Pictures</th>
      <th>production_company_Paramount Pictures</th>
      <th>production_company_Twentieth Century Fox Film Corporation</th>
      <th>production_company_Columbia Pictures</th>
      <th>production_company_Metro-Goldwyn-Mayer (MGM)</th>
      <th>production_company_New Line Cinema</th>
      <th>production_company_Touchstone Pictures</th>
      <th>production_company_Walt Disney Pictures</th>
      <th>production_company_Columbia Pictures Corporation</th>
      <th>production_company_TriStar Pictures</th>
      <th>production_company_Relativity Media</th>
      <th>production_company_Canal+</th>
      <th>production_company_United Artists</th>
      <th>production_company_Miramax Films</th>
      <th>production_company_Village Roadshow Pictures</th>
      <th>production_company_Regency Enterprises</th>
      <th>production_company_BBC Films</th>
      <th>production_company_Dune Entertainment</th>
      <th>production_company_Working Title Films</th>
      <th>production_company_Fox Searchlight Pictures</th>
      <th>production_company_StudioCanal</th>
      <th>production_company_Lionsgate</th>
      <th>production_company_DreamWorks SKG</th>
      <th>production_company_Fox 2000 Pictures</th>
      <th>production_company_Summit Entertainment</th>
      <th>production_company_Hollywood Pictures</th>
      <th>production_company_Orion Pictures</th>
      <th>production_company_Amblin Entertainment</th>
      <th>production_company_Dimension Films</th>
      <th>num_countries</th>
      <th>production_country_United States of America</th>
      <th>production_country_United Kingdom</th>
      <th>production_country_France</th>
      <th>production_country_Germany</th>
      <th>production_country_Canada</th>
      <th>production_country_India</th>
      <th>production_country_Italy</th>
      <th>production_country_Japan</th>
      <th>production_country_Australia</th>
      <th>production_country_Russia</th>
      <th>production_country_Spain</th>
      <th>production_country_China</th>
      <th>production_country_Hong Kong</th>
      <th>production_country_Ireland</th>
      <th>production_country_Belgium</th>
      <th>production_country_South Korea</th>
      <th>production_country_Mexico</th>
      <th>production_country_Sweden</th>
      <th>production_country_New Zealand</th>
      <th>production_country_Netherlands</th>
      <th>production_country_Czech Republic</th>
      <th>production_country_Denmark</th>
      <th>production_country_Brazil</th>
      <th>production_country_Luxembourg</th>
      <th>production_country_South Africa</th>
      <th>num_languages</th>
      <th>language_English</th>
      <th>language_Français</th>
      <th>language_Español</th>
      <th>language_Deutsch</th>
      <th>language_Pусский</th>
      <th>language_Italiano</th>
      <th>language_日本語</th>
      <th>language_普通话</th>
      <th>language_हिन्दी</th>
      <th>language_</th>
      <th>language_Português</th>
      <th>language_العربية</th>
      <th>language_한국어/조선말</th>
      <th>language_广州话 / 廣州話</th>
      <th>language_தமிழ்</th>
      <th>language_Polski</th>
      <th>language_Magyar</th>
      <th>language_Latin</th>
      <th>language_svenska</th>
      <th>language_ภาษาไทย</th>
      <th>language_Český</th>
      <th>language_עִבְרִית</th>
      <th>language_ελληνικά</th>
      <th>language_Türkçe</th>
      <th>language_Dansk</th>
      <th>language_Nederlands</th>
      <th>language_فارسی</th>
      <th>language_Tiếng Việt</th>
      <th>language_اردو</th>
      <th>language_Română</th>
      <th>num_cast</th>
      <th>cast_name_Samuel L. Jackson</th>
      <th>cast_name_Robert De Niro</th>
      <th>cast_name_Morgan Freeman</th>
      <th>cast_name_J.K. Simmons</th>
      <th>cast_name_Bruce Willis</th>
      <th>cast_name_Liam Neeson</th>
      <th>cast_name_Susan Sarandon</th>
      <th>cast_name_Bruce McGill</th>
      <th>cast_name_John Turturro</th>
      <th>cast_name_Forest Whitaker</th>
      <th>cast_name_Willem Dafoe</th>
      <th>cast_name_Bill Murray</th>
      <th>cast_name_Owen Wilson</th>
      <th>cast_name_Nicolas Cage</th>
      <th>cast_name_Sylvester Stallone</th>
      <th>genders_0_cast</th>
      <th>genders_1_cast</th>
      <th>genders_2_cast</th>
      <th>cast_character_</th>
      <th>cast_character_Himself</th>
      <th>cast_character_Herself</th>
      <th>cast_character_Dancer</th>
      <th>cast_character_Additional Voices (voice)</th>
      <th>cast_character_Doctor</th>
      <th>cast_character_Reporter</th>
      <th>cast_character_Waitress</th>
      <th>cast_character_Nurse</th>
      <th>cast_character_Bartender</th>
      <th>cast_character_Jack</th>
      <th>cast_character_Debutante</th>
      <th>cast_character_Security Guard</th>
      <th>cast_character_Paul</th>
      <th>cast_character_Frank</th>
      <th>num_crew</th>
      <th>crew_name_Avy Kaufman</th>
      <th>crew_name_Robert Rodriguez</th>
      <th>crew_name_Deborah Aquila</th>
      <th>crew_name_James Newton Howard</th>
      <th>crew_name_Mary Vernieu</th>
      <th>crew_name_Steven Spielberg</th>
      <th>crew_name_Luc Besson</th>
      <th>crew_name_Jerry Goldsmith</th>
      <th>crew_name_Francine Maisler</th>
      <th>crew_name_Tricia Wood</th>
      <th>crew_name_James Horner</th>
      <th>crew_name_Kerry Barden</th>
      <th>crew_name_Bob Weinstein</th>
      <th>crew_name_Harvey Weinstein</th>
      <th>crew_name_Janet Hirshenson</th>
      <th>genders_0_crew</th>
      <th>genders_1_crew</th>
      <th>genders_2_crew</th>
      <th>jobs_Producer</th>
      <th>jobs_Executive Producer</th>
      <th>jobs_Director</th>
      <th>jobs_Screenplay</th>
      <th>jobs_Editor</th>
      <th>jobs_Casting</th>
      <th>jobs_Director of Photography</th>
      <th>jobs_Original Music Composer</th>
      <th>jobs_Art Direction</th>
      <th>jobs_Production Design</th>
      <th>jobs_Costume Design</th>
      <th>jobs_Writer</th>
      <th>jobs_Set Decoration</th>
      <th>jobs_Makeup Artist</th>
      <th>jobs_Sound Re-Recording Mixer</th>
      <th>departments_Production</th>
      <th>departments_Sound</th>
      <th>departments_Art</th>
      <th>departments_Crew</th>
      <th>departments_Writing</th>
      <th>departments_Costume &amp; Make-Up</th>
      <th>departments_Camera</th>
      <th>departments_Directing</th>
      <th>departments_Editing</th>
      <th>departments_Visual Effects</th>
      <th>departments_Lighting</th>
      <th>departments_Actors</th>
      <th>log_budget</th>
      <th>has_homepage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>828</th>
      <td>3829</td>
      <td>0</td>
      <td>NaN</td>
      <td>tt0210130</td>
      <td>en</td>
      <td>Jails, Hospitals &amp; Hip-Hop</td>
      <td>Jails, Hospitals &amp;amp; Hip-Hop is a cinematic ...</td>
      <td>0.009057</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>90.0</td>
      <td>NaN</td>
      <td>three worlds / two million voices / one genera...</td>
      <td>Jails, Hospitals &amp; Hip-Hop</td>
      <td>{}</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Drama</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
test.loc[test['release_date'].isnull() == True, 'release_date'] = '05/01/00'
```


```python
train['release_date'] = train['release_date'].apply(lambda x: fix_date(x))
test['release_date'] = test['release_date'].apply(lambda x: fix_date(x))

```

 

### Task 4: Creating Features Based on Release Date


```python
train['release_date'] = pd.to_datetime(train['release_date'])
test['release_date'] = pd.to_datetime(test['release_date'])

```


```python
def process_date(df):
    date_parts = ['year', 'weekday', 'month', 'weekofyear', 'day', 'quarter']
    for part in date_parts:
        part_col = 'release_date' + '_' + part
        df[part_col] = getattr(df['release_date'].dt, part).astype(int)
    return df

train = process_date(train)
test = process_date(test)
```

 

### Task 5: Using Plotly to Visualize the Number of Films Per Year


```python
d1 = train['release_date_year'].value_counts().sort_index()
d2 = test['release_date_year'].value_counts().sort_index()

```


```python
import plotly.offline as py
py.init_notebook_mode(connected = True)
import plotly.graph_objs as go

data = [go.Scatter(x=d1.index, y=d1.values, name='train'), 
        go.Scatter(x=d1.index, y=d2.values, name='test')]

layout = go.Layout(dict(title = 'Number of films per year', 
                        xaxis = dict(title = 'Year'),
                        yaxis = dict(title = 'Count'),
                        ), legend = dict(orientation='v'))

py.iplot(dict(data=data, layout=layout))

```


<script type="text/javascript">
window.PlotlyConfig = {MathJaxConfig: 'local'};
if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
if (typeof require !== 'undefined') {
require.undef("plotly");
requirejs.config({
    paths: {
        'plotly': ['https://cdn.plot.ly/plotly-latest.min']
    }
});
require(['plotly'], function(Plotly) {
    window._Plotly = Plotly;
});
}
</script>




<div>


            <div id="ad850cb1-0819-4d09-9e5d-19ce4c5a95d6" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("ad850cb1-0819-4d09-9e5d-19ce4c5a95d6")) {
                    Plotly.newPlot(
                        'ad850cb1-0819-4d09-9e5d-19ce4c5a95d6',
                        [{"name": "train", "type": "scatter", "x": [1921, 1924, 1925, 1926, 1927, 1928, 1930, 1931, 1932, 1933, 1935, 1936, 1938, 1939, 1940, 1942, 1943, 1944, 1945, 1947, 1948, 1949, 1950, 1951, 1952, 1953, 1954, 1955, 1956, 1957, 1958, 1959, 1960, 1961, 1962, 1963, 1964, 1965, 1966, 1967, 1968, 1969, 1970, 1971, 1972, 1973, 1974, 1975, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985, 1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017], "y": [1, 1, 2, 1, 2, 3, 1, 2, 3, 2, 1, 4, 1, 4, 2, 3, 2, 2, 4, 2, 6, 2, 2, 4, 1, 6, 6, 7, 4, 3, 3, 5, 8, 4, 5, 5, 6, 4, 7, 8, 7, 6, 8, 8, 8, 9, 5, 8, 7, 11, 14, 16, 21, 26, 25, 27, 30, 42, 44, 45, 56, 47, 46, 42, 45, 56, 51, 62, 55, 63, 66, 65, 65, 77, 84, 74, 83, 99, 114, 105, 101, 106, 126, 124, 125, 141, 123, 128, 125, 40]}, {"name": "test", "type": "scatter", "x": [1921, 1924, 1925, 1926, 1927, 1928, 1930, 1931, 1932, 1933, 1935, 1936, 1938, 1939, 1940, 1942, 1943, 1944, 1945, 1947, 1948, 1949, 1950, 1951, 1952, 1953, 1954, 1955, 1956, 1957, 1958, 1959, 1960, 1961, 1962, 1963, 1964, 1965, 1966, 1967, 1968, 1969, 1970, 1971, 1972, 1973, 1974, 1975, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985, 1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017], "y": [1, 1, 4, 1, 1, 2, 1, 1, 2, 4, 3, 4, 2, 2, 2, 2, 3, 6, 3, 5, 2, 4, 3, 3, 4, 6, 6, 5, 8, 3, 9, 4, 5, 5, 6, 12, 10, 10, 8, 8, 8, 16, 16, 9, 15, 12, 8, 14, 14, 18, 15, 18, 29, 24, 38, 39, 42, 53, 48, 52, 59, 67, 76, 55, 58, 74, 71, 83, 73, 72, 83, 95, 86, 94, 91, 99, 115, 106, 126, 137, 167, 153, 165, 178, 180, 187, 180, 194, 197, 184, 175, 58, 1]}],
                        {"legend": {"orientation": "v"}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Number of films per year"}, "xaxis": {"title": {"text": "Year"}}, "yaxis": {"title": {"text": "Count"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('ad850cb1-0819-4d09-9e5d-19ce4c5a95d6');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


 

### Task 6: Number of Films and Revenue Per Year


```python
d1 = train['release_date_year'].value_counts().sort_index()
d2 = train.groupby(['release_date_year'])['revenue'].sum()

data = [go.Scatter(x=d1.index, y=d1.values, name='film count'), 
        go.Scatter(x=d1.index, y=d2.values, name='total revenue', yaxis='y2')]

layout = go.Layout(dict(title = 'Number of films and total revenue per year', 
                        xaxis = dict(title = 'Year'),
                        yaxis = dict(title = 'Count'),
                        yaxis2 = dict(title = 'Total revenue', overlaying='y', side='right')), 
                   legend = dict(orientation='v'))

py.iplot(dict(data=data, layout=layout))
```


<div>


            <div id="3588ca3b-677d-49ce-86bb-6c9c4f4e4aa1" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("3588ca3b-677d-49ce-86bb-6c9c4f4e4aa1")) {
                    Plotly.newPlot(
                        '3588ca3b-677d-49ce-86bb-6c9c4f4e4aa1',
                        [{"name": "film count", "type": "scatter", "x": [1921, 1924, 1925, 1926, 1927, 1928, 1930, 1931, 1932, 1933, 1935, 1936, 1938, 1939, 1940, 1942, 1943, 1944, 1945, 1947, 1948, 1949, 1950, 1951, 1952, 1953, 1954, 1955, 1956, 1957, 1958, 1959, 1960, 1961, 1962, 1963, 1964, 1965, 1966, 1967, 1968, 1969, 1970, 1971, 1972, 1973, 1974, 1975, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985, 1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017], "y": [1, 1, 2, 1, 2, 3, 1, 2, 3, 2, 1, 4, 1, 4, 2, 3, 2, 2, 4, 2, 6, 2, 2, 4, 1, 6, 6, 7, 4, 3, 3, 5, 8, 4, 5, 5, 6, 4, 7, 8, 7, 6, 8, 8, 8, 9, 5, 8, 7, 11, 14, 16, 21, 26, 25, 27, 30, 42, 44, 45, 56, 47, 46, 42, 45, 56, 51, 62, 55, 63, 66, 65, 65, 77, 84, 74, 83, 99, 114, 105, 101, 106, 126, 124, 125, 141, 123, 128, 125, 40]}, {"name": "total revenue", "type": "scatter", "x": [1921, 1924, 1925, 1926, 1927, 1928, 1930, 1931, 1932, 1933, 1935, 1936, 1938, 1939, 1940, 1942, 1943, 1944, 1945, 1947, 1948, 1949, 1950, 1951, 1952, 1953, 1954, 1955, 1956, 1957, 1958, 1959, 1960, 1961, 1962, 1963, 1964, 1965, 1966, 1967, 1968, 1969, 1970, 1971, 1972, 1973, 1974, 1975, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985, 1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017], "y": [2500000, 1213880, 45101, 966878, 1027878, 1517097, 7940, 3239189, 3194025, 5460000, 3202000, 15855000, 4000000, 15252757, 17000000, 18833500, 2806624, 21663000, 44380386, 7952961, 19194420, 3821349, 5063463, 53050000, 55240, 165609651, 73914313, 114134595, 156500000, 41150000, 8263404, 197775000, 123821000, 48700000, 76829846, 140881636, 198444549, 173335864, 96215984, 279413945, 135988444, 162897395, 265640872, 276722404, 326867674, 580304491, 295501369, 723843036, 517828656, 592379651, 396158759, 894247728, 703253992, 917556227, 939182796, 485716248, 1167705089, 1340612547, 1655786468, 1666637476, 1828559485, 3048102302, 2619875459, 1128866274, 2617733845, 3065410881, 2545295849, 4649121280, 3109042085, 4098978931, 3551313039, 5054548104, 4124784418, 5812507144, 7373002194, 5840168441, 6665570199, 8108901129, 7131686267, 8051234684, 8175452857, 8204226430, 7855118086, 9017211264, 10770751722, 10208471341, 9432880004, 13293335806, 9348124945, 7256157404], "yaxis": "y2"}],
                        {"legend": {"orientation": "v"}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Number of films and total revenue per year"}, "xaxis": {"title": {"text": "Year"}}, "yaxis": {"title": {"text": "Count"}}, "yaxis2": {"overlaying": "y", "side": "right", "title": {"text": "Total revenue"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('3588ca3b-677d-49ce-86bb-6c9c4f4e4aa1');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>



```python
d1 = train['release_date_year'].value_counts().sort_index()
d2 = train.groupby(['release_date_year'])['revenue'].mean()

data = [go.Scatter(x=d1.index, y=d1.values, name='film count'), 
        go.Scatter(x=d1.index, y=d2.values, name='mean revenue', yaxis='y2')]

layout = go.Layout(dict(title = 'Number of films and average revenue per year', 
                        xaxis = dict(title = 'Year'),
                        yaxis = dict(title = 'Count'),
                        yaxis2 = dict(title = 'Avearge revenue', overlaying='y', side='right')), 
                   legend = dict(orientation='v'))

py.iplot(dict(data=data, layout=layout))
```


<div>


            <div id="75ce9921-a73b-4a9b-84b7-a07c868a3e62" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("75ce9921-a73b-4a9b-84b7-a07c868a3e62")) {
                    Plotly.newPlot(
                        '75ce9921-a73b-4a9b-84b7-a07c868a3e62',
                        [{"name": "film count", "type": "scatter", "x": [1921, 1924, 1925, 1926, 1927, 1928, 1930, 1931, 1932, 1933, 1935, 1936, 1938, 1939, 1940, 1942, 1943, 1944, 1945, 1947, 1948, 1949, 1950, 1951, 1952, 1953, 1954, 1955, 1956, 1957, 1958, 1959, 1960, 1961, 1962, 1963, 1964, 1965, 1966, 1967, 1968, 1969, 1970, 1971, 1972, 1973, 1974, 1975, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985, 1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017], "y": [1, 1, 2, 1, 2, 3, 1, 2, 3, 2, 1, 4, 1, 4, 2, 3, 2, 2, 4, 2, 6, 2, 2, 4, 1, 6, 6, 7, 4, 3, 3, 5, 8, 4, 5, 5, 6, 4, 7, 8, 7, 6, 8, 8, 8, 9, 5, 8, 7, 11, 14, 16, 21, 26, 25, 27, 30, 42, 44, 45, 56, 47, 46, 42, 45, 56, 51, 62, 55, 63, 66, 65, 65, 77, 84, 74, 83, 99, 114, 105, 101, 106, 126, 124, 125, 141, 123, 128, 125, 40]}, {"name": "mean revenue", "type": "scatter", "x": [1921, 1924, 1925, 1926, 1927, 1928, 1930, 1931, 1932, 1933, 1935, 1936, 1938, 1939, 1940, 1942, 1943, 1944, 1945, 1947, 1948, 1949, 1950, 1951, 1952, 1953, 1954, 1955, 1956, 1957, 1958, 1959, 1960, 1961, 1962, 1963, 1964, 1965, 1966, 1967, 1968, 1969, 1970, 1971, 1972, 1973, 1974, 1975, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985, 1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017], "y": [2500000.0, 1213880.0, 22550.5, 966878.0, 513939.0, 505699.0, 7940.0, 1619594.5, 1064675.0, 2730000.0, 3202000.0, 3963750.0, 4000000.0, 3813189.25, 8500000.0, 6277833.333333333, 1403312.0, 10831500.0, 11095096.5, 3976480.5, 3199070.0, 1910674.5, 2531731.5, 13262500.0, 55240.0, 27601608.5, 12319052.166666666, 16304942.142857144, 39125000.0, 13716666.666666666, 2754468.0, 39555000.0, 15477625.0, 12175000.0, 15365969.2, 28176327.2, 33074091.5, 43333966.0, 13745140.57142857, 34926743.125, 19426920.57142857, 27149565.833333332, 33205109.0, 34590300.5, 40858459.25, 64478276.777777776, 59100273.8, 90480379.5, 73975522.28571428, 53852695.54545455, 28297054.214285713, 55890483.0, 33488285.333333332, 35290624.115384616, 37567311.84, 17989490.666666668, 38923502.96666667, 31919346.35714286, 37631510.63636363, 37036388.35555556, 32652847.94642857, 64853240.4680851, 56953814.32608695, 26877768.42857143, 58171863.222222224, 54739480.01785714, 49907761.74509804, 74985827.09677419, 56528037.90909091, 65063157.634920634, 53807773.31818182, 77762278.52307692, 63458221.81538462, 75487105.76623377, 87773835.64285715, 78921195.14864865, 80308074.68674698, 81908092.21212122, 62558651.46491228, 76678425.56190476, 80945077.79207921, 77398362.5471698, 62342207.03174603, 72719445.67741935, 86166013.776, 72400505.964539, 76690081.33333333, 103854185.984375, 74784999.56, 181403935.1], "yaxis": "y2"}],
                        {"legend": {"orientation": "v"}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Number of films and average revenue per year"}, "xaxis": {"title": {"text": "Year"}}, "yaxis": {"title": {"text": "Count"}}, "yaxis2": {"overlaying": "y", "side": "right", "title": {"text": "Avearge revenue"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('75ce9921-a73b-4a9b-84b7-a07c868a3e62');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


 

### Task 7: Do Release Days Impact Revenue?


```python
sns.catplot(x='release_date_weekday', y='revenue', data = train);
plt.title('Reverue on different days of the week')

```




    Text(0.5, 1.0, 'Reverue on different days of the week')




    
![png](output_56_1.png)
    



```python

```

 

### Task 8: Relationship between Runtime and Revenue


```python
sns.distplot(train['runtime'].fillna(0)/60, bins=40, kde=False);
plt.title('Distribution of the length of films in hours');
```


    
![png](output_60_0.png)
    



```python
sns.scatterplot(train['runtime'].fillna(0)/60, train['revenue'])
plt.title('Runtime vs Revenue')
```




    Text(0.5, 1.0, 'Runtime vs Revenue')




    
![png](output_61_1.png)
    


 

 
