# User Classification Flywheel Project
This project is designed to classify and monitor user engagement dynamics over time across our platform. It leverages predictive models to label user sessions and aggregate user behaviors into flywheel engagement states. The workflow is structured around processing rolling date windows and producing labeled datasets to support downstream analysis and visualization.

## Overview
The process consists of the following main components:

1. Session Quality Prediction

   - Each user session is scored using a trained classification model.

   - Input features include impressions, pageviews, detail pageviews, favorites, shares, leads, and categorical counts.

   - The model outputs a probability distribution over session quality labels (e.g., High Quality, Medium Quality, Very Low Quality).

2. User-Level Aggregation

   - Predicted session labels are grouped by user over a given time window.

   - The counts of each session label per user are normalized into a distribution.

   - A secondary classification model then predicts the user’s flywheel engagement label based on this distribution of session types.

3. Rolling Window Evaluation

   - The pipeline iterates over fixed time windows (e.g., 2-week intervals).

   - For each window:

     - Sessions are filtered to the relevant date range.

     - Extreme outliers are trimmed from numeric features using quantile thresholds.

     - Session and user predictions are generated and merged.

     - Results are exported in parquet format, labeled with interval metadata.

4. Timestamps and Intervals

   - Each record is enriched with derived timestamps:

     - Unix timestamp extracted from the session ID.

     - Converted UTC and MST datetimes.

   - Interval identifiers and interval date ranges are assigned to facilitate downstream reporting.

# Inputs
   - Data Sources:

     - A parquet file containing all session records within the target date range.
  
     - Pre-trained models:

       - session_quality_model.keras

       - user_flywheel_model.keras

     - Scalers and encoders:

       - minmax_scaler_sessions.pkl

       - minmax_scaler_users.pkl

       - label_encoder_sessions.pkl

       - label_encoder_users.pkl

# Outputs
  For each processed interval, the pipeline produces:

  - Session-level parquet file containing:

    - All original metrics and features.

    - Predicted session quality label.

    - Predicted user flywheel label.

    - Timestamps in UTC and MST.

    - Interval identifier and interval date string.

  - User-level parquet file (optional) containing:

    - Aggregated user labels for the interval.

# Key Processing Steps
  1. Quantile Trimming

      - For each numeric feature, the top 0.1% of values are excluded to reduce the impact of outliers.

  2. Feature Scaling

      - Session-level features are scaled using the saved MinMaxScaler.

      - User-level distributions are scaled with their respective scaler to match the training setup.

  3. Prediction

      - The session model outputs probabilities and predicted classes.

      - Predicted session labels are converted back to their string representation.

      - User-level label distributions are fed to the user model to generate flywheel classifications.

  4. Metadata Enrichment

      - Timestamps are parsed from session IDs.

      - Interval metadata columns (Interval and Interval_Dates) are attached.

# Use Cases
  These outputs are primarily used to:

  - Monitor shifts in user engagement over time.

  - Feed Looker Studio dashboards (e.g., time series charts of flywheel label distributions).

  - Support cohort analyses of movement between engagement states.

  - Provide labeled datasets for experimentation and modeling.

# Notes
Sankey Charts in Looker Studio:
  - To visualize user label transitions over time, you will need to pre-aggregate label transitions into a table of From_Label, To_Label, and User_Count. Looker Studio does not pivot session-level data dynamically into this structure.

Timezone Handling:
  - All timestamps are stored both as UTC and MST for convenience in reporting.

Interval Labeling:
  - Each export is tagged with the interval number and date range so files can be merged or joined later without confusion.
