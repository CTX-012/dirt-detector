# NOVA: floor-cleaning robot and perception code

NOVA is a floor-cleaning robot. Its camera looks at the floor, finds objects (dirt, hair, spills, paper,
socks and textiles, cables and wires, pet waste), and decides for each one whether to **CLEAN** it,
**AVOID** it, or **ALERT** someone. The robot then plans and drives its cleaning coverage.

This repository holds **242 Python files** written for that work. To stay under GitHub's 100-file upload
limit, the code is packed into **6 zip files**, one per part of the project. The `.md` files here explain
what is inside each zip without opening it.

## What's in this repository

```
README.md                 ← this file: what each zip holds and how it's laid out
docs/
  CLASS_GUIDE.md          ← each floor object (hair, dirt, paper, …): what the robot does + which code handles it
  CODE_GUIDE.md           ← every .py file inside every zip, with its path and what it does
  HOW_TO_RUN.md           ← unzip, install, run the tests, launch the robot, run the dirt benchmark
zips/
  01_perception_kit_classes.zip
  02_nova_robot_ros2.zip
  03_dirt_detector_shootout.zip
  04_robot_tests_obstacle_speed.zip
  05_nova_apps_vision_voice_social.zip
  06_home_scripts.zip
```

## What's inside each zip

Every zip also contains `README.md` and `CLASS_INDEX.md` (which files mention which object class).

### `01_perception_kit_classes.zip` · 32 `.py` + 5 small data files the tests read · 101 KB
The perception tool kit: the code that prepares data for the six object classes, trains and scores models, and checks safety rules.
```
01_perception_kit_classes/
  kit/                      dataset tools + their tests
    inspect_dataset.py        open a downloaded dataset before training: flags fake class names, missing labels, missing licence
    obb_to_seg.py             rotated-box labels → segmentation polygons
    relabel_locked6.py        map any dataset onto the 6 classes (drops labels, never images)
    make_train_view.py        the copy that's actually trained on (stops the trainer reading un-remapped labels)
    hair_split.py / hair_ingest.py   hair mask labelling: fixed held-out split, then rule-checked import
    ssl_pretrain.py           self-supervised pretraining of the YOLO11n backbone on floor images
    score_real.py             real-floor scoring: hit = mask IoU ≥ 0.3; false-safe = an AVOID object called CLEAN
    paired_compare.py         compare two models on the same objects (McNemar test, confidence intervals)
    cal_ingest.py, rig_session_check.py, migrate_policy.py, g8_agreement.py, sync_pending_to_hdd.py
    test_*.py                 one test file per tool (120 tests)
    cal2.schema.json, cal2.template.json, rig_sheet.template.json, README.md   calibration templates + kit guide
  contract_hazard_report/hazard/hazard_log.yaml   the hazard table the hazard tests read
  contract_hazard_report/
    contract/contract_check.py   checks the robot-facing perception messages against the interface contract
    hazard/hazard_plr.py         hazard log → required ISO 13849-1 safety level
    build_report_html.py         renders the main report to HTML
  tracker_tools/backfill_map.py  maps experiment-tracker rows to tasks
  run_arms.sh, run_all_tests.sh  train+score both model versions / run every test
```

### `02_nova_robot_ros2.zip` · 52 `.py` · 135 KB
The robot itself, as ROS 2 (Humble) packages.
```
02_nova_robot_ros2/
  nova/nova/                cleaning behaviour, one ROS 2 node per file
    world_cleaner.py, room_cleaner.py, explore_cleaner.py, auto_cleaner.py   whole-world / per-room / exploring / roller-aligned cleaning
    dirt_finder.py, dirt_detector.py, dirt_mapper.py, dirt_manager.py, dirt_cleaner.py, dirt_spawner.py   find, map and clean dirt (+ spawn test dirt in sim)
    coverage_planner.py, coverage_mapper.py, coverage_tracker.py, room_segmenter.py   coverage planning and room splitting
    mission.py, docker.py, vacuum_pump.py, switch_floor_server.py, switch_floor_client.py      missions, docking, pump, changing floors
    map_relay.py, odom_tf_broadcaster.py, range_converter.py                          map/TF/sensor plumbing
  nova/launch/              14 launch files: SLAM, navigation, room/world clean, floor 2, hospital world, dashboard
  nova_coverage_tracker/    a separate package that tracks which floor area has been cleaned (+ its test)
  cleaner_bot_description/launch/   an earlier robot model: display, SLAM, Nav2, exploration
```

### `03_dirt_detector_shootout.zip` · 33 `.py` · 116 KB
A benchmark of dirt-detection methods, scored on recall at a 5% false-positive rate, plus a desktop demo.
```
03_dirt_detector_shootout/dirt/
  scripts/
    shootout.py, shootout_harness.py, verify_shootout.py   run and verify the comparison
    scorers/   histogram_diff, deep_hist, spatial_diff, saliency_sr, padim, gmm_novelty, dinov2_diff, vlm_smol
    benchmark.py, make_indoor_benchmark.py, make_synthetic.py, video_test.py
    download_indoor.py, download_kaggle.py, download_roboflow.py   dataset download helpers (keys come from env vars)
  nova_desktop_pkg/         desktop app + nova_perception/ (dirt detector, geometry, coverage control, cleaning verification)
  webapp/app.py             demo web page: upload an image or video, see detections
```

### `04_robot_tests_obstacle_speed.zip` · 19 `.py` · 37 KB
Checks used in simulation test runs of the robot.
```
04_robot_tests_obstacle_speed/nova_tests/
  check_abort.py, check_freeze.py, check_climb.py, check_termination.py   abort / freeze / climbing onto obstacles / stop rule
  check_waypoints.py, check_wp_fail.py, check_chunks.py, check_contract.py   waypoint and plan checks
  measure_speed.py, measure_dutycycle.py, clock_rate.py, inflation.py   speed, duty cycle, sim real-time factor, costmap inflation
  mission_report.py, run_report.py, smoke.py, capture_detect_view.py, grab_frames.py, decompose.py, window.py
```

### `05_nova_apps_vision_voice_social.zip` · 72 `.py` · 195 KB
Applications built around the robot.
```
05_nova_apps_vision_voice_social/
  nova-vision-bot-v5/   camera vision bot (robot/ + scripts/)
  nova_voice_gate/      speech-to-text voice gate on the GPU
  nova-social/backend/  receptionist kiosk: camera, vision, speech-to-text, text-to-speech, host relay
  nova-ai/              LLM-driven robot control (scripts, launch, ROS 2 workspace)
  nova/                 LLM and VLM clients, memory system, logger, sentinel, void detector (stops the robot when LiDAR sees > 3 m in every direction, i.e. it is lost in open space)
  nova_backup/          an older backup copy of some robot nodes (newer versions are in zip 02)
```

### `06_home_scripts.zip` · 34 `.py` · 116 KB
Standalone scripts from early experiments: AI mapping, auto-navigation, autonomous exploration, LLM robot control.
```
06_home_scripts/home/   ai_mapper.py, auto_nav.py, autonomous_explorer.py, llm_robot.py, nova_sentinel.py, …
```

## Not in this repository, on purpose

- **Datasets, images and model weights.** Weights trained on public data are for benchmarking only and are not distributed.
- **Code that is already on GitHub elsewhere:** the robot-vacuum coverage stack (`CTX-012/Rob`) and `CTX-012/nova-vlm-1bit`.
- **Third-party code** (AWS RoboMaker worlds, m-explore-ros2, rf2o_laser_odometry, Isaac Sim, Blender) and backup `.bak` copies.
- **No API keys.** Scripts read them from environment variables. The only key-like values in the code are `your_key_here` placeholders.

The paths inside the code point at the original development machine. Change them before running. See `docs/HOW_TO_RUN.md`.
