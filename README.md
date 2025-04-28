# data-anomaly-detector

Creating a complete Python program for an anomaly detection system for streaming data involves several key steps. Here's a simple implementation that uses the Z-score method for anomaly detection. This system will read streaming data, detect anomalies, and alert when an anomaly is detected. 

We will use the `pandas` library for data handling and the `scipy.stats` for calculating the Z-score. For the sake of this example, we'll simulate streaming data using a generator function.

Here's a complete implementation:

```python
import pandas as pd
import numpy as np
import random
from scipy.stats import zscore

def generate_streaming_data(num_points=1000):
    """Simulate a generator function to stream data."""
    for _ in range(num_points):
        # Normal data with occasional spikes
        yield random.gauss(10, 2) if random.random() > 0.05 else random.gauss(50, 5)

class DataAnomalyDetector:
    def __init__(self, threshold=3.0, window_size=100):
        """
        Initialize the anomaly detector.
        
        Parameters:
        - threshold: The Z-score threshold above which a point is considered an anomaly.
        - window_size: The number of data points to consider for calculating the Z-score.
        """
        self.threshold = threshold
        self.window_size = window_size
        self.data_window = []
        
    def update_window(self, new_value):
        """Update the data window with a new value and maintain the window size."""
        if len(self.data_window) >= self.window_size:
            self.data_window.pop(0)
        self.data_window.append(new_value)
        
    def detect_anomaly(self, new_value):
        """Detect anomalies using Z-score method."""
        self.update_window(new_value)
        
        if len(self.data_window) < self.window_size:
            # Not enough data to detect anomalies
            return False, None
        
        # Calculate the Z-score for the current window
        z_scores = zscore(self.data_window)
        
        # Check the Z-score of the new value
        if abs(z_scores[-1]) > self.threshold:
            return True, z_scores[-1]
        
        return False, z_scores[-1]
    
    def alert(self, anomaly, value, z_score):
        """Generate an alert if an anomaly is detected."""
        if anomaly:
            print(f"Anomaly detected! Value: {value}, Z-score: {z_score:.2f}")

def main():
    try:
        detector = DataAnomalyDetector(threshold=3.0, window_size=30)
        stream = generate_streaming_data(num_points=500)
        
        for data_point in stream:
            anomaly, z_score = detector.detect_anomaly(data_point)
            detector.alert(anomaly, data_point, z_score)
    
    except Exception as e:
        print(f"An error occurred: {e}")

if __name__ == "__main__":
    main()
```

### Explanation:

1. **Simulating Streaming Data**: The `generate_streaming_data` function simulates a data stream, producing normally distributed data with occasional spikes to mimic anomalies.

2. **DataAnomalyDetector Class**: 
   - Responsible for managing a sliding window of the most recent data points.
   - Calculates the Z-score for the data in the window to recognize statistical anomalies.
   - Checks if the latest data point's Z-score exceeds the threshold, flagging it as an anomaly.

3. **Anomaly Detection**:
   - We maintain a window of recent data. If a new data point has a Z-score higher than the specified threshold, it is flagged as an anomaly.

4. **Alerting**:
   - When an anomaly is detected, an alert is printed. This can be expanded to send emails, log to a file, or trigger other notification systems.

5. **Error Handling**:
   - Any exceptions that occur during execution are caught and printed using a try-except block.

This basic framework can be expanded with features such as more sophisticated alert mechanisms, persistence storage for detected anomalies, or integration with real-time data streams.