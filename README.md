# ESP32-CAM Onion Object Recognition with Edge Impulse

![Project workflow](images/project-workflow.png)

This repository documents the supplied workflow for collecting onion images using an AI-Thinker ESP32-CAM, training an object-detection model in Edge Impulse, exporting an Arduino library, and running the model on the ESP32-CAM. The steps are kept in the same order as the original process. Interface names and library APIs may vary by version.

## Contents
- [1. Project overview](#1-project-overview)
- [2. Hardware required](#2-hardware-required)
- [3. Circuit design and connections](#3-circuit-design-and-connections)
- [4. Software setup](#4-software-setup)
- [5. Install Eloquent ESP32-CAM](#5-install-eloquent-esp32-cam)
- [6. Configure Wi-Fi and camera pinout](#6-configure-wi-fi-and-camera-pinout)
- [7. Upload the image-collection sketch](#7-upload-the-image-collection-sketch)
- [8. Open the image-collection server](#8-open-the-image-collection-server)
- [9. Collect around 50 images](#9-collect-around-50-images)
- [10. Upload images to Edge Impulse](#10-upload-images-to-edge-impulse)
- [11. Label the images](#11-label-the-images)
- [12. Design the impulse](#12-design-the-impulse)
- [13. Generate features](#13-generate-features)
- [14. Train the model](#14-train-the-model)
- [15. Deploy the Arduino library](#15-deploy-the-arduino-library)
- [16. Configure and upload inference code](#16-configure-and-upload-inference-code)
- [17. Test object detection](#17-test-object-detection)
- [18. Troubleshooting](#18-troubleshooting)

## 1. Project overview

The ESP32-CAM is used to collect images of an onion from the camera that will be installed in the final project. These images are uploaded to Edge Impulse, labelled with bounding boxes, and used to train an object-detection model. The trained model is exported as an Arduino library and integrated with an ESP32 camera example. Detection results, including a confidence score when provided by the example, are viewed in the Serial Monitor.

An OLED is optional in this process. Displaying detections on an OLED requires additional display code and compatible GPIO wiring.

## 2. Hardware required

- AI-Thinker ESP32-CAM with camera module
- FTDI USB-to-TTL programmer
- Jumper wires
- Stable 5 V supply suitable for ESP32-CAM current demand
- USB cable for FTDI
- Optional I²C OLED display
- Onion for dataset collection and testing

## 3. Circuit design and connections

![ESP32-CAM to FTDI circuit diagram](images/esp32cam-ftdi-circuit.png)

### ESP32-CAM ↔ FTDI wiring

| AI-Thinker ESP32-CAM | FTDI programmer | Function |
|---|---|---|
| 5V | 5V | Power |
| GND | GND | Common ground |
| U0R / GPIO3 (RX) | TX | Serial data into ESP32-CAM |
| U0T / GPIO1 (TX) | RX | Serial data out of ESP32-CAM |
| GPIO0 | GND during flashing only | Serial bootloader mode |

**Important:** GPIO0 must be **LOW (connected to GND)** while entering programming mode. Once the upload has finished, remove the GPIO0-to-GND connection and reset or power-cycle the board so the sketch boots normally. Do not leave GPIO0 grounded for normal operation.

Use a stable 5 V supply. Check the FTDI voltage selector, board pin labels, and wiring before powering the circuit. Camera and Wi-Fi activity can cause current spikes.

### Optional OLED

The OLED is not required for image collection, training, or serial inference testing. Its wiring depends on the display interface and the GPIO pins available after camera connections. Confirm the AI-Thinker pin mapping and avoid camera or boot-configuration pins. Use Serial Monitor for the initial test.

## 4. Software setup

1. Install Arduino IDE: <https://www.arduino.cc/en/software>
2. Create a free account and open Edge Impulse Studio: <https://studio.edgeimpulse.com/>
3. Open the Eloquent ESP32-CAM library: <https://github.com/eloquentarduino/EloquentEsp32cam>
4. Install ESP32 board support in Arduino IDE if it is not already installed.

## 5. Install Eloquent ESP32-CAM

1. Open Arduino IDE.
2. Open **Sketch → Include Library → Manage Libraries** (Library Manager).
3. Search for `EloquentEsp32Cam` / `Eloquent ESP32Cam`, depending on the listing.
4. Install the library and any dependencies it requires.
5. Open **File → Examples → EloquentEsp32Cam**.
6. Select **Collect_Images_for_EdgeImpulse**.

Example names may vary by library version. Use the collection example included with your installed library.

## 6. Configure Wi-Fi and camera pinout

Open the image-collection example.

1. Find the Wi-Fi SSID setting and enter your network name.
2. Find the password setting and enter your Wi-Fi password.
3. Find the camera pinout selection.
4. Replace the WROOM-S3 selection with the AI-Thinker ESP32-CAM pinout.

The intended change is similar to:

```cpp
// Previous example selection:
// camera.pinout.wroom_s3();

// AI-Thinker pinout (if supported by your installed library):
camera.pinout.aithinker();
```

The exact API depends on the installed Eloquent ESP32-CAM version. If `camera.pinout.aithinker()` does not compile, use the AI-Thinker selection documented by your installed version. Do not use a WROOM-S3 pin map for an AI-Thinker board.

## 7. Upload the image-collection sketch

1. Connect the ESP32-CAM to FTDI according to the wiring table.
2. Connect GPIO0 to GND to enter flashing mode.
3. Select the appropriate ESP32 board and COM port in Arduino IDE.
4. Click **Upload** and wait for it to finish.
5. Disconnect GPIO0 from GND.
6. Reset or power-cycle the board.
7. Open **Tools → Serial Monitor** and select the baud rate specified by the sketch, often 115200.

The supplied workflow resets the board after flashing and checks Serial Monitor for Wi-Fi connection information and the collection server's IP address.

## 8. Open the image-collection server

1. Wait for the ESP32-CAM to connect to Wi-Fi.
2. Read the IP address printed in Serial Monitor.
3. Connect the computer or phone to the same local network.
4. Open Chrome and enter the IP address in the address bar.
5. Confirm that the image collection interface appears.

Use the IP printed by your own board. The address varies by network.

## 9. Collect around 50 images

1. Place the ESP32-CAM in the same approximate position, angle, and distance intended for the final project.
2. Open the collection server in Chrome.
3. Start image collection.
4. Show the onion in different orientations and positions.
5. Capture around **50 images** for the initial dataset.
6. When finished, stop collection and save/download the images using the interface's available controls.

Try to include useful variation in orientation, distance, lighting, and background. Avoid collecting only nearly identical frames. Images from the target ESP32-CAM help match the final camera's image characteristics, although another camera can also be used to collect training images.

## 10. Upload images to Edge Impulse

1. Open your Edge Impulse project.
2. Go to **Data acquisition**.
3. Click **Add data** or the image-upload option.
4. Select the folder containing the captured images.
5. Choose the files for upload.
6. Assign the label `onion` if prompted.
7. Use the automatic train/test split option if it is available.
8. Upload the data.
9. Repeat the folder/file upload process **three times**, following the supplied workflow, until all image batches have been uploaded.

After upload, check that all intended images are present and assigned to the appropriate dataset split. Uploading in three batches is the described process; the number of batches itself does not improve accuracy.

## 11. Label the images

After upload, the images may appear in **Labeling queue**, because object detection needs bounding-box annotations.

1. Open **Labeling queue**.
2. Select an image.
3. In label suggestions, choose **Track object between frames**, as in the supplied workflow, if the option is available.
4. Select the onion and assign the class `onion`.
5. Draw or adjust a bounding box around the onion.
6. Save the annotation.
7. Repeat for every image containing an onion until the labeling queue is empty.

Review each suggested box. Tracking suggestions may not be accurate for independent still images; the important outcome is a correct onion bounding box for every image containing an onion.

## 12. Design the impulse

1. Open **Impulse design**.
2. Add an **Image** processing block.
3. Set image dimensions to **96 × 96**.
4. Choose **Fit shortest axis** for resizing.
5. Set the learning block to **Object Detection**.
6. Save the impulse.

Available learning blocks and supported image sizes can vary by Edge Impulse model. Keep the settings compatible with the selected learning block and deployment example.

## 13. Generate features

1. Open the **Image** processing block.
2. Change the colour mode to **Grayscale**, following the supplied workflow for a low-cost development board.
3. Click **Save parameters**.
4. Click **Generate features**.
5. Inspect the feature/cluster graph.

The cluster graph shows how examples are represented by the processing pipeline. Greater separation between classes can be encouraging, but it does not prove real-world accuracy. Also review model metrics and unseen test images.

## 14. Train the model

1. Open the **Object Detection** neural-network settings.
2. Set **Number of training cycles** to **60**.
3. The supplied notes mention `0.001` as a good starting learning rate, then use **`0.01`** for this run to try to reduce training time.
4. Select **CPU** as the training processor if that option is available.
5. Start training and wait for the output.

A learning rate of `0.01` is an experiment, not a guarantee of faster or more accurate training. If the loss is unstable or results are poor, compare with the recommended/default learning rate. Training time depends on the model, dataset, and platform.

After training, review the evaluation metrics and test images before deployment.

## 15. Deploy the Arduino library

1. Open **Deployment**.
2. Configure the deployment block.
3. Select **Arduino library**.
4. In the target selection, choose **Espressif ESP-EYE**, following the supplied workflow because a separate ESP32-CAM target may not be listed.
5. Set the target device/options shown by Studio.
6. Click **Build**.
7. Download the generated Arduino library ZIP file.

**Compatibility note:** ESP-EYE and AI-Thinker ESP32-CAM are different hardware designs. Selecting ESP-EYE for export does not remove the need to configure the AI-Thinker camera pin map in the Arduino example and verify compatibility with the generated library.

## 16. Configure and upload inference code

1. In Arduino IDE, select **Sketch → Include Library → Add .ZIP Library**.
2. Select the downloaded Edge Impulse library ZIP.
3. Go to **File → Examples** and locate the example generated for your project. Its name may resemble `*_detect_interfacing`, depending on the library.
4. Open the ESP32 camera inference example.
5. Find the camera-model definitions.
6. Comment out the ESP-EYE definition and enable the AI-Thinker definition.

The intended configuration is similar to:

```cpp
// #define CAMERA_MODEL_ESP_EYE
#define CAMERA_MODEL_AI_THINKER  // Has PSRAM
```

Use the exact macro spelling present in your example. Some versions use a different camera configuration mechanism.

Then:

1. Connect GPIO0 to GND for flashing.
2. Select the correct ESP32 board and COM port.
3. Compile and upload the inference sketch.
4. Disconnect GPIO0 from GND after upload.
5. Reset or power-cycle the board.
6. Open Serial Monitor at the baud rate specified by the example.

## 17. Test object detection

1. Place an onion in front of the ESP32-CAM.
2. Check the Serial Monitor output.
3. Confirm that the model reports the `onion` class.
4. Observe the confidence score or percentage printed by the example.
5. Test different onion orientations and distances.
6. Test non-onion objects and empty backgrounds to check for false detections.

A score of `0.85` means the model assigned a score of 85% to that particular detection; it does **not** mean the model has 85% overall accuracy. Test on images not used during training to evaluate generalisation.

## 18. Troubleshooting

| Problem | Checks |
|---|---|
| Upload does not start | GPIO0-to-GND during flashing, correct COM port, crossed TX/RX, reset timing |
| Board resets or disconnects | Stable 5 V supply, common ground, adequate current |
| Wi-Fi does not connect | SSID/password, signal strength, network availability |
| Collection IP is missing | Serial Monitor baud rate and Wi-Fi connection messages |
| Browser interface does not load | Correct current IP address and same local network |
| AI-Thinker pinout method does not compile | Installed library version and supported pinout API |
| Camera initialisation fails | Correct AI-Thinker camera definition and pin mapping |
| No onion is detected | Labels, bounding boxes, image resolution, model metrics, camera configuration |
| Arduino library fails to compile | Compatible Edge Impulse library, ESP32 board package, and example |
| Low detection quality | More varied images, corrected annotations, and unseen test data |

## Useful links

- [Arduino IDE](https://www.arduino.cc/en/software)
- [Edge Impulse Studio](https://studio.edgeimpulse.com/)
- [Eloquent ESP32-CAM library](https://github.com/eloquentarduino/EloquentEsp32cam)

## License

Choose a license for your repository before publishing. Review the licenses of all third-party libraries used by the project.
