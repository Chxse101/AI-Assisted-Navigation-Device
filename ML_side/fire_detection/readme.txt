================================================================================
         FIRE DETECTION SYSTEM — README
         Real-Time Fire Intensity Comparison & Performance Evaluation
================================================================================

OVERVIEW
--------
This project uses a webcam to detect and compare the intensity of two fire
sources in real time using computer vision (OpenCV + HSV colour thresholding).
It also includes a full performance metrics evaluation pipeline so you can
measure how accurately the system detects fire.

The project consists of THREE notebooks:

  1. fire_compare.ipynb                    — Original live detection (no logging)
  2. fire_compare_with_logging.ipynb       — Live detection WITH data logging
  3. fire_detection_metrics_realtime.ipynb — Metrics evaluation on real data

--------------------------------------------------------------------------------

FILES IN THIS PROJECT
---------------------

- fire_compare.ipynb: Original fire detection notebook. Captures two fire frames via webcam, compares their intensity scores, and announces the result via text-to-speech. No data is saved.

- fire_compare_with_logging.ipynb: Updated version of fire_compare.ipynb. Works identically but also saves every captured frame, its predicted mask, intensity score, and your manual ground-truth label (fire / no-fire) into a file called fire_session_data.npz. Use this instead of the original when you want to evaluate performance metrics afterward.

- fire_detection_metrics_realtime.ipynb: Loads fire_session_data.npz and computes a full set of performance metrics based on your real webcam predictions and manual labels. Produces a multi-panel dashboard chart and a threshold sensitivity plot.

- fire_detection_metrics.py: Standalone Python script version of the metrics evaluator. Uses a synthetic test set (no webcam required). Useful for quick testing without needing to capture real frames.

- fire_session_data.npz: Auto-generated when you run fire_compare_with_logging.ipynb and press q to quit. Contains all captured frames, masks, scores, labels, and timestamps from your session. Required by the metrics notebook.


--------------------------------------------------------------------------------

RECOMMENDED WORKFLOW
---------------------

  STEP 1 — Capture real fire data
      Open and run:  fire_compare_with_logging.ipynb

      Controls during the webcam session:
        c  — Capture the current frame
             (you will be asked in the terminal: fire or no-fire?)
        r  — Reset the comparison (clears frame 1 and frame 2)
        s  — Save session data immediately (without quitting)
        q  — Quit and auto-save all captured data

      Tips for good data:
        - Capture at least 10-20 fire frames and 10-20 no-fire frames
        - Vary the distance, angle, and brightness of the fire
        - Include some borderline frames (small flames, reflections) to
          test how the system handles edge cases

      Output: fire_session_data.npz  (saved in the same folder)

  STEP 2 — Evaluate performance metrics
      Place fire_detection_metrics_realtime.ipynb in the same folder as
      fire_session_data.npz, then open and run all cells.

      The notebook will output:
        - Full metrics report printed to the cell output
        - Dashboard chart saved as fire_realtime_dashboard.png
        - Threshold sensitivity plot saved as fire_threshold_sweep.png
        - The best classification threshold for your specific data

  STEP 3 — Tune and repeat
      If metrics are low, adjust the HSV colour ranges in detect_fire_mask()
      or update the THRESHOLD value in Cell 3 of the metrics notebook to the
      recommended value printed by the threshold sweep. Then re-capture and
      re-evaluate.

--------------------------------------------------------------------------------

METRICS EXPLAINED
-----------------

  CLASSIFICATION METRICS (frame-level, fire vs no-fire)
  -------------------------------------------------------
  Accuracy      Percentage of frames classified correctly overall.
  Precision     Of all frames predicted as fire, how many actually were fire.
                High precision = few false alarms.
  Recall        Of all actual fire frames, how many were detected.
                High recall = few missed fires.
  Specificity   Of all no-fire frames, how many were correctly identified.
  F1 Score      Harmonic mean of Precision and Recall. Best single metric
                when you care about both false alarms and missed detections.
  MCC           Matthews Correlation Coefficient. Reliable even when fire and
                no-fire frame counts are unequal.
  ROC-AUC       Area under the ROC curve. 1.0 = perfect, 0.5 = random.
  Avg Precision Area under the Precision-Recall curve. Useful when fire frames
                are much rarer than no-fire frames.

  SEGMENTATION METRICS (pixel-level, fire region accuracy)
  ---------------------------------------------------------
  Mean IoU      Intersection over Union. Measures how well the predicted fire
                mask overlaps with the actual fire pixels. 1.0 = perfect.
  Mean Dice     Similar to IoU but weights overlap more heavily. Also 0-1.

  SPEED METRICS
  -------------
  FPS (mask)    Frames per second for the HSV masking step alone.
  FPS (full)    Frames per second for the complete intensity calculation.
  Latency (ms)  Average time per frame in milliseconds.

  CONFUSION MATRIX
  ----------------
  TP  True Positive  — fire frame correctly detected as fire
  TN  True Negative  — no-fire frame correctly detected as no-fire
  FP  False Positive — no-fire frame wrongly detected as fire (false alarm)
  FN  False Negative — fire frame missed by the detector (dangerous!)

--------------------------------------------------------------------------------

REQUIREMENTS
------------

  Python        3.10 or higher (3.10+ required for the X | Y type hints)
  opencv-python For webcam capture and HSV processing
  numpy         Array operations
  pyttsx3       Text-to-speech announcements (fire_compare notebooks only)
  scikit-learn  Metrics computation
  seaborn       Heatmap visualisation
  matplotlib    All charts and plots

  Install all at once:
      pip install opencv-python numpy pyttsx3 scikit-learn seaborn matplotlib

  Each notebook also contains a pip install cell at the top that runs
  automatically — so you can also just open and run the notebook directly.

--------------------------------------------------------------------------------

HOW THE FIRE DETECTION WORKS
-----------------------------

  1. Each frame from the webcam is converted from BGR colour space to HSV
     (Hue, Saturation, Value), which separates colour from brightness and
     makes fire colours easier to isolate.

  2. Two HSV colour ranges are thresholded to create a binary fire mask:
       Range 1: Hue 0-35,  Saturation 120-255, Value 200-255  (orange/red fire)
       Range 2: Hue 35-60, Saturation 50-255,  Value 200-255  (yellow fire)

  3. Morphological operations (OPEN then DILATE) clean up the mask by
     removing small noise pixels and filling gaps in the fire region.

  4. An intensity score (0.0 to ~1.0) is computed as a weighted sum of:
       50%  Area ratio     — how much of the frame is fire
       20%  Brightness     — how bright the fire pixels are
       20%  Hot pixel ratio — proportion of very orange/red pixels
       10%  Flicker        — pixel difference between the current and previous
                             frame (more movement = more intense fire)

  5. The two captured frames are compared by intensity score and the result
     is announced verbally (e.g. "First fire is more intense, take route 2").

--------------------------------------------------------------------------------

NOTES & LIMITATIONS
--------------------

  - The HSV thresholds are tuned for typical candle/flame colours under normal
    indoor lighting. Performance may vary with different fire types, lighting
    conditions, or camera white balance settings.

  - The system works best when the fire fills a reasonable portion of the
    frame. Very small flames at distance may not trigger detection.

  - Bright orange or yellow objects (e.g. traffic cones, orange clothing,
    sunlight reflections) may produce false positives. Increase the
    Saturation lower bound or narrow the Hue range to reduce this.

  - pyttsx3 requires a working audio output device. If you get errors,
    you can comment out all speak() calls without affecting detection.

  - For best metrics results, aim for a balanced dataset: roughly equal
    numbers of fire and no-fire frames in your captured session.

--------------------------------------------------------------------------------

AUTHOR NOTES
------------

  This system was built and evaluated in a Jupyter notebook environment.
  The core detection logic lives in detect_fire_mask() and calculate_intensity()
  and is identical across all three notebooks, so any improvements you make
  to those functions will automatically be reflected in the metrics when you
  re-capture and re-evaluate.

================================================================================