# Feature Engineering Documentation for Spotify Hit Prediction Model

This document details how each column from the original dataset was handled in the final model.

## Original Features Used As-Is

### Audio Features
1. **danceability** (float)
   - Used directly
   - Already normalized between 0 and 1
   - Represents how suitable a track is for dancing

2. **energy** (float)
   - Used directly
   - Already normalized between 0 and 1
   - Represents the intensity and activity of the track

3. **loudness** (float)
   - Used directly
   - In decibels (dB)
   - No normalization needed as it's a standard unit

4. **speechiness** (float)
   - Used directly
   - Already normalized between 0 and 1
   - Detects presence of spoken words

5. **acousticness** (float)
   - Used directly
   - Already normalized between 0 and 1
   - Confidence measure of whether the track is acoustic

6. **instrumentalness** (float)
   - Used directly
   - Already normalized between 0 and 1
   - Predicts whether a track contains no vocals

7. **liveness** (float)
   - Used directly
   - Already normalized between 0 and 1
   - Detects presence of an audience in the recording

8. **valence** (float)
   - Used directly
   - Already normalized between 0 and 1
   - Musical positiveness measure

9. **tempo** (float)
   - Used directly
   - In beats per minute (BPM)
   - No normalization needed as it's a standard unit

## Engineered Features

### Duration Transformation
1. **duration_ms** → **duration_minutes_log**
   - Original: Duration in milliseconds
   - Transformation: 
     1. Converted to minutes (`duration_ms / 60000`)
     2. Applied log transformation (`np.log1p`)
   - Reason: 
     - Log transformation helps handle the right-skewed distribution of song durations
     - Makes the feature more normally distributed
     - Reduces the impact of extreme outliers

### Composite Audio Features
1. **energy_loudness_composite**
   - Formula: `energy * loudness.clip(lower=0)`
   - Reason: 
     - Captures the combined effect of energy and loudness
     - Clipping negative loudness values prevents negative composites
     - Helps capture the perceived intensity of the track

2. **energy_acousticness_ratio**
   - Formula: `energy / (acousticness + 0.01)`
   - Reason:
     - Captures the contrast between electronic and acoustic elements
     - Added 0.01 to prevent division by zero
     - Higher values indicate more electronic/produced tracks

3. **danceability_energy**
   - Formula: `danceability * energy`
   - Reason: Captures the combined effect of dance-friendliness and track intensity

4. **valence_energy**
   - Formula: `valence * energy`
   - Reason: Captures the relationship between positiveness and track intensity

5. **loudness_danceability**
   - Formula: `loudness * danceability`
   - Reason: Captures the relationship between volume and dance-friendliness

### Musical Theory Features
1. **key** → **key_encoded**
   - Original: Integer (0-11)
   - Used directly as it's already numerically encoded
   - Represents the key of the track using standard pitch class notation

2. **mode**
   - Original: Binary (0 or 1)
   - Used directly
   - `1` for major key, `0` for minor key
   - Already in optimal format for modeling

4. **time_signature** → **time_signature_encoded**
   - Used directly as it's already numerically encoded
   - Represents the time signature of the track

### Categorical Features with Label Encoding

1. **track_genre** → **genre_label_encoded**
   - Transformation: Label encoding
   - Reason:
     - Avoids data leakage that would occur with mean encoding
     - Better than one-hot encoding due to large number of genres
     - Maintains genre distinctness without introducing target information

2. **artists** → **artist_label_encoded**
   - Transformation: Label encoding
   - Reason:
     - Avoids data leakage that would occur with mean encoding
     - Better than one-hot encoding due to large number of artists
     - Maintains artist distinctness without introducing target information

3. **album_name** → **album_label_encoded**
   - Transformation: Label encoding
   - Reason:
     - Avoids data leakage that would occur with mean encoding
     - Better than one-hot encoding due to large number of albums
     - Maintains album distinctness without introducing target information

### Boolean Features
1. **explicit** → **is_explicit**
   - Original: Boolean
   - Transformation: Converted to integer (0 or 1)
   - Reason: Most ML models work better with numeric data

## Columns Not Used

1. **track_id**
   - Reason: Unique identifier, no predictive value
   - Type: String

2. **track_name**
   - Reason: 
     - Unique identifier
     - Would require complex NLP techniques for meaningful features
     - High cardinality makes it impractical for direct encoding
   - Type: String

## Target Variable

**popularity** (integer)
- Used as the target variable
- Scale: 0 to 100
- No transformation applied as it's already well-distributed

## Notes on Encoding Choices

1. **Mean Encoding** was chosen for high-cardinality categorical variables (genre, artists, album) because:
   - One-hot encoding would create too many features
   - Label encoding would impose arbitrary ordering
   - Mean encoding captures meaningful relationships with the target variable

2. **Binary Encoding** was used for boolean and binary categorical variables because:
   - Simple and effective for binary data
   - Maintains interpretability
   - No information loss

3. **Direct Numeric Use** was chosen for pre-encoded features (key, time_signature) because:
   - Already in numeric format
   - Values have inherent meaning in music theory
   - No benefit from additional encoding

4. **Log Transformation** was used for duration because:
   - Handles right-skewed distribution
   - Makes the relationship with target more linear
   - Reduces impact of outliers