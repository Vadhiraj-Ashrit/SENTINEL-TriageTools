### Head Injury Criterion (HIC)

The HIC originated as a modified version of the Generalized Acceleration Severity Index (GSI), which limited the time interval over which GSI was evaluated. It was developed based on experiments measuring the effects of acceleration on the intracranial pressure of dogs, stemming from the works of Gurdjian and collaborators (Gurdjian et al., 1953, 1964, 1966). The concept of HIC was first proposed by Versace (1971), who noted the necessity of comparing acceleration pulses to the WSTC using the average acceleration pulse of interest. The expression for HIC was subsequently modified by the NHTSA to provide a better comparison with long-duration human volunteer tests (Kleinberger et al., 1998). The modified HIC expression is:

$$HIC=\left[ \left[ \frac{1}{[t_2-t_1]}\int_{t_1}^{t_2}a(t) \, dt \right]^{2.5}(t_2-t_1) \right]_{max}$$

Where, $a(t)$ is the magnitude of linear head acceleration measured in g. The resultant acceleration (measured in g) is of duration $T$, and $t1$ and $t2$ are two time points (measured in milliseconds) during the impact, such that $(0 < t_1 < t_2 < T)$. HIC considers head acceleration over time, with a lower probability of injury associated with shorter acceleration duration. Optimisation of HIC is achieved by varying the difference $(t_2-t_1)$. Specific variants such as $({HIC}_{36})$  and $({HIC}_{15})$ are calculated with differences set to 36 milliseconds and 15 milliseconds, respectively. A HIC of 1000 corresponds to a 16\% probability of AIS4+ head injury, while a HIC of 1500 could result in a 56% probability of AIS4+. It is important to note that HIC is a non-injury-specific criterion and does not directly relate to injury severity. Additionally, it does not account for variations in brain mass or loading direction (Hutchinson et al., 1998).

---

### How the Code Works

This script calculates the **Head Injury Criterion (HIC)** from acceleration data recorded over time. HIC is a critical measure used to evaluate head impact severity in fields like automotive safety and sports science. The script involves data preparation, filtering, and calculating HIC values iteratively over different time windows to find the maximum HIC value and its corresponding time range.

---

### Workflow Explanation

1. **Command-Line Arguments**  
   - The script takes arguments via the command line using `argparse`. These arguments define:
     - Sampling frequency of the data (`--frequency`).
     - Path to the CSV file containing acceleration data (`--file_path`).
     - Column indices for X, Y, and Z acceleration components (`--x_location`, `--y_location`, `--z_location`).

   Example:
   ```bash
   python hic_calculator.py --frequency 0.00001 --file_path "data.csv" --x_location 2 --y_location 3 --z_location 4
   ```

---

2. **Initialize `HICCalculator`**  
   - An instance of the `HICCalculator` class is created using the provided frequency. [1]
   - Default values for the Butterworth low-pass filter are also initialized (`cutoff = 1650 Hz`, `order = 2`). [2]

---

3. **Read and Process the Data**  
   - The script reads the CSV file from the provided file path.  
   - It calculates the magnitude of acceleration for each timestamp using the formula: $magnitude = \sqrt{x^2 + y^2 + z^2} / 9810$, The division by `9810` converts the magnitude to G-forces.  
   - The raw magnitude data is passed through a Butterworth low-pass filter to remove noise and high-frequency components.

   **Key Operations in This Step**:
   - **Time Calculation**: The time for each row is calculated based on the sampling frequency.  
     Example: `time = row_index * frequency`.
   - **Filtering**: The magnitudes are filtered using the `scipy.signal.butter` and `scipy.signal.lfilter` functions.

---

4. **Iterative HIC Calculation**  
   - The script calculates HIC for various time windows. It starts with a window size equal to the sampling frequency and iterates up to `15 ms`.  
   - For each window, the HIC value is calculated using the formula:
     \[
     HIC = \left(\frac{1}{t_2 - t_1} \int_{t_1}^{t_2} a(t) \, dt \right)^{2.5} (t_2 - t_1)
     \]
     where \( t_1 \) and \( t_2 \) are the start and end times of the window, and \( a(t) \) is the filtered acceleration magnitude.

   **Steps**:
   - Calculate the integral of acceleration over the window.
   - Compute the HIC value for the current window.
   - Store the HIC value and the time window in a dictionary.

---

5. **Find Maximum HIC**  
   - After iterating through all possible time windows up to `15 ms`, the script determines the maximum HIC value and the corresponding time window.

---

6. **Output the Results**  
   - The maximum HIC value and its time range are printed to the console:
     ```
     The HIC 15 value is <max_hic_value> and was achieved between the time window of <max_hic_window>.
     ```

---

### Example Workflow

**Input CSV** (Sample rows):
```csv
X,Y,Z
0.1,0.2,0.3
0.15,0.25,0.35
0.2,0.3,0.4
...
```

**Steps**:
1. User specifies the file path and column indices for X, Y, Z.  
2. The script computes time and acceleration magnitude for each row.
3. Magnitudes are filtered using a low-pass filter.
4. HIC is calculated for multiple overlapping time windows.
5. The script outputs the maximum HIC value and the time window.

**Output Example**:
```
The HIC 15 value is 123.45 and was achieved between the time window of 0.05:0.07.
```

---

### Flow Chart

```plaintext
  +-------------------------------------+
  | Start                               |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | Parse arguments                     |
  | (frequency, file_path, x, y, z)     |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | Initialize HICCalculator            |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | Read and process CSV data           |
  | (calculate time, magnitude, filter) |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | Initialize HIC calculation loop     |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | For each time window (0 - 15ms):    |
  | - Calculate HIC                     |
  | - Store HIC value and time window   |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | Find max HIC and corresponding      |
  | time window                         |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | Output max HIC and time window      |
  +-------------------------------------+
               |
               v
  +-------------------------------------+
  | End                                 |
  +-------------------------------------+
```
---
### Reference
1. Kleinberger, M., Sun, E., Eppinger, R., Kuppa, S., & Saul, R. (1998). Development of improved injury criteria for the assessment of advanced automotive restraint systems. NHTSA Docket 4405.9, 12–17. https://www.nhtsa.gov/sites/nhtsa.gov/files/criteria_0.pdf
2. https://help.oasys-software.com/articles/#!this-20-0/standard-sae-filter-options
3. Gurdjian, E. S., Hodgson, V. R., Hardy, W. G., Patrick, L. M., & Lissner, H. R. (1964). Evaluation of the protective characteristics of helmets in sports. Journal of Trauma - Injury, Infection and Critical Care, 4 (3), 309–324. https://doi.org/10.1097/00005373-196405000-00005
4. Gurdjian, E. S., Lissner, H. R., Latimer, F. R., Haddad, B. F., & Webster, J. E. (1953). Quantitative determination of acceleration and intracranial pressure in experimental head injury: Preliminary report. Neurology, 3 (6), 417–423. https://doi.org/10.1212/wnl.3.6.417
5. Gurdjian, E. S., Roberts, V. L., & Thomas, L. M. (1966). Tolerance curves of acceleration and intracranial pressure and protective index in experimental head injury. Journal of Trauma - Injury, Infection and Critical Care, 6 (5), 600–604. https://doi.org/10.1097/00005373-196609000-00005
6. Versace, J. (1971). A Review of the Severity Index. SAE Technical Paper 710881. https://doi.org/10.4271/710881
