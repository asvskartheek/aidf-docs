# Suspicious Activity Detection Service

A vision pipeline AI service that analyzes person detection results (person tracks) to generate Suspicious Activity (SA) annotations based on visible keypoint counts. A post-processing module that works on top of people detection output — no ML model inference required.

!!! info "Location"
    **Code directory:** `Code/Suspicious_Activity/`
    **Version:** 1.0.0
    **Created:** December 22, 2025

## Detection Logic

Classifies occlusion levels based on the number of visible body keypoints:

| Level | Name | Keypoints Visible |
|-------|------|-------------------|
| **Level 0** | Fully Visible | 15–17 keypoints |
| **Level 1** | Partly Occluded | 10–14 keypoints |
| **Level 2** | Largely Occluded | 5–9 keypoints |
| **Skipped** | Too Occluded | <5 keypoints |

## Processing Pipeline

```
1. API receives request
   Flask endpoint with JSON payload including storage config and video file paths

2. Download person tracks JSON
   For each input video, download the corresponding video_id.json from configured storage

3. Submit SA detection job
   Uses Ray actor pattern for parallel processing across multiple videos

4. Analyze keypoints per frame
   For each person per frame, count visible keypoints → classify occlusion level

5. Generate SA annotation JSON
   Aggregate per-frame results with statistics and upload to target storage
```

## Output JSON Structure

```json
{
  "success": true,
  "detection_type": "people_suspicious_activity",
  "video_info": {
    "video_id": "video1",
    "video_metadata": { "resolution": "1920x1080", "fps": 30.0, "total_frames": 900 }
  },
  "configuration": {
    "occlusion_levels": {
      "level_0": "15-17 keypoints (fully visible)",
      "level_1": "10-14 keypoints (partly occluded)",
      "level_2": "5-9 keypoints (largely occluded)"
    }
  },
  "summary": {
    "total_persons_processed": 150,
    "total_sa_detections": 120,
    "too_occluded_skipped": 30,
    "detection_rate": 80.0
  }
}
```

## Ray Integration

The service uses **Ray** for distributed, parallel execution:

- Configure `enable_ray: Y` to use Ray cluster for parallel processing
- Uses **Actor pattern** for stateful, persistent processing across video batches
- `ray_address`: Connect to an existing cluster or start a new one
- `ray_task_num_cpus`: Control CPU allocation per task (default: 1)

## Deployment

```bash
docker build -t suspicious-activity:1.0 .
docker run -p 8002:8002 -e ENVIRONMENT=Development suspicious-activity:1.0

# Endpoint:
POST /api/<aiservices>/<centific>/loop/model/inference/suspicious_activity_detection
```
