# Stop Sign Behavior Annotation Protocol

This guide defines how to label driver behavior near stop signs in dash-cam videos.  
You will assign **three labels per clip**:  
1. **Stop Behavior**  
2. **Sign Visibility**  
3. **Confidence Score**  

Assume you are the driver of the recording car (the *ego vehicle*).

---

## 1. Labeling Instructions for Annotators

### Stop Behavior (pick **one**)

| Label           | Definition                                                                                |
|-----------------|-------------------------------------------------------------------------------------------|
| `full_stop`     | Ego vehicle is completely motionless for **≥ 0.5 s** (15 frames @ 30 fps). Scene freezes. |
| `rolling_stop`  | Vehicle slows noticeably but never fully stops.                                           |
| `no_stop`       | Vehicle maintains speed through the intersection (no visible deceleration).               |
| `uncertain`     | Behavior can’t be judged (occlusion, glare, camera shake, etc.).                          |

> **Tip:** Judge motion by watching poles, trees, and road texture. If unsure, see edge case examples in Appendix 4.1.

### Stop Sign Visibility (pick **one**)

| Label               | Definition                                                  |
|---------------------|-------------------------------------------------------------|
| `visible`           | Entire stop sign is clearly in frame.                       |
| `partially_visible` | Only part of the sign is seen, yet still identifiable.      |
| `not_visible`       | No stop sign appears anywhere in the video.                 |

### Confidence Score (pick **one**)

| Label      | When to Use                                                                 |
|------------|------------------------------------------------------------------------------|
| `high`     | You are certain of both behavior and sign visibility.                       |
| `medium`   | You are mostly confident, but a small doubt remains (e.g., lighting, angle). |
| `low`      | You are unsure but selected the best label you could.                       |

> Use `low` instead of guessing (this helps with quality control and weighting during model training).

### Annotation Workflow

1. Watch the clip once from beginning to end.  
2. Find the first frame where the stop sign appears.  
3. Observe the ego vehicle’s motion until it passes the intersection.  
4. Assign the following labels:  
   - **Stop Behavior**  
   - **Sign Visibility**  
   - **Confidence Score**

---

## 2. Assumptions & Trade-Offs

1. **Visual inference only**: no GPS, speed, or telemetry  
2. **0.5 s stop threshold**: clear legal cutoff for labeling a stop  
3. **Behavior labeled only when stop sign is visible**  
4. **External context (pedestrians, traffic) is ignored**  
5. **`uncertain` is allowed** when confidence is too low  
6. **Front-facing dashcam view assumed** across all clips  

---

## 3. Model Justification of Labels

* `Stop Behavior`: provides the primary supervised target for detecting compliance with stop signs  
* `Sign Visibility`: ensures the model is trained only on behavior occurring in proper context  
* `Confidence Score`: allows dynamic label weighting (high-confidence samples can be prioritized during training or used for evaluation, while low-confidence ones can be down-weighted or reviewed) 
* `uncertain` labels can be omitted or used for semi-supervised learning techniques  
* The labeling schema is intentionally simple and structured, maximizing agreement between annotators.

---

## 4. Appendix

### 4.1 Edge-Cases 

This subsection lists tricky or ambiguous scenarios annotators may encounter. Use these guidelines to ensure consistent labeling. When in doubt, choose `Uncertain` and note the reason.

#### Visibility Edge Cases

| Scenario | Labeling Guidance |
|----------|-------------------|
| **Stop sign is blocked by a parked car or tree** | Label `Partially_Visible` if any part of the sign is recognizable; otherwise `Not_Visible`. |
| **Stop sign is very small or far ahead** | If it can be identified as a stop sign, use `Partially_Visible`. |
| **Sign is visible only for 2–3 frames** | Still label `Visible` or `Partially_Visible` if it's recognizable. If unsure, use `Uncertain`. |
| **Temporary stop sign on construction stand** | Treat as valid. Label `Visible` or `Partially_Visible` as applicable. |
| **Stop line or pole is visible but no sign** | Label `Not_Visible`. Do not infer intent from markings. |

#### Behavior Ambiguities

| Scenario | Labeling Guidance |
|----------|-------------------|
| **Camera shakes slightly but scene mostly still** | If the car appears motionless (road/poles don’t shift), label `Full_Stop`. |
| **Driver stops behind another vehicle** | Still label `Full_Stop` if ego vehicle’s scene freezes for 0.5 s or more. |
| **Clip cuts off before stop or sign is passed** | Label `Uncertain`. Do not guess if stop was made. |
| **Driver stops before the stop sign** | If it's clearly related to the stop sign, label normally. If unrelated (e.g., pedestrian crossing earlier), label `Uncertain`. |
| **Driver stops due to pedestrian or traffic** | Still label `Full_Stop` if ego vehicle is motionless ≥ 0.5 s (stop cause does not matter). |

#### Environmental Factors

If environmental conditions (night footage, heavy glare, rain, snow, or fog) obscure key visual details, label the clip as `Uncertain`.

---

### 4.2 Example Clip Labels

| Clip ID        | Stop Behavior  | Sign Visibility     | Confidence Score | Notes                          |
|----------------|----------------|----------------------|------------|---------------------------------|
| `3e120c7f-d830-4ee9-b9fb-f3e2781b8e7e.mp4` | `full_stop`    | `visible`            | `high`     | Clear full stop, good lighting |
| `8af4bb81-e8b3-43bc-a75c-c74e568336ae.mp4` | `rolling_stop` | `visible`  | `high`   |  Scene does not freeze  |
| `0561132f-a92b-49f5-bcc9-3fd82737dd5a.mp4` | `uncertain`      | `visible`            | `medium`     | Accident occurs in front of vehicle at stop sign        |
| `b68b2d97-c4f4-4d61-88e5-c5c7ef0ff000.mp4` | `no_stop`    | `visible`        | `high`      | No sign of deceleration |
