# Train-YoloV4-on-Custom-Dataset

**Requirements and steps:-**

    Clone YoloV4: !git clone https://github.com/AlexeyAB/darknet
  
    1.) Annotated Data
        A) clone:> git clone https://github.com/yash-007/Train-YoloV5-on-Custom-Dataset.git
        B) change dir:> cd Train-YoloV5-on-Custom-Dataset.git
        C) train:> python main.py downloader --classes "Vehicle registration plate" --type_csv train --limit 1000
        D) val:> python main.py downloader --classes "Vehicle registration plate" --type_csv validation --limit 200
        E) edit classes.txt
        F) python convert_annotations.py
        G) train/Vehicle registration plate => train/obj, validation/Vehicle registration plate => validation/test
        H) zip both obj and test folders and upload to drive in a folder named 'yolov4'
        I) copy both folders from drive to colab session: !cp /mydrive/yolov4/obj.zip ../ and !cp /mydrive/yolov4/test.zip ../
        J) unzip both in /darknet/data/ => !unzip ../obj.zip -d data/ and !unzip ../test.zip -d data/
    
    2.) Custom .cfg file

        A) !cp cfg/yolov4-custom.cfg /mydrive/yolov4/yolov4-obj.cfg
        B) edit yolov4-obj.cfg and make the following changes accordingly:
          for training:
            batch=64
            subdivisions=16 #32 if any problem
          width=416
          height=416
          max_batches=(num_classes)*2000 (min is 6000)
          steps=80% of max_batches, 90% of max_batches
          classes=1
          filters (b/w conv and yolo layer) = (num_classes+5)*3
          3 occurances of both classes and filters to be changed
        C) copy back to cfg folder, !cp /mydrive/yolov4/yolov4-obj.cfg ./cfg 

    3.) obj.data and obj.names files

        A) create 'obj.names' file inside yolov4 folder. 
           It should have 1 class name per line in the same order as our 'classes.txt'. 
          (You can rename them, but the order should be same)
        B) create 'obj.data' file in yolov4 folder. It will point to the image and annotation files.

            classes = 1
            train = data/train.txt
            valid = data/test.txt
            names = data/obj.names
            backup = /mydrive/yolov4/backup/
          
        C) copy these 2 files to darknet/data folder.
            !cp /mydrive/yolov4/obj.names ./data
            !cp /mydrive/yolov4/obj.data ./data

    4.) train.txt file

        copy 'create_train.py' and 'create_test.py' to /darknet/data folder.
        !cp /mydrive/yolov4/create_train.py ./
        !cp /mydrive/yolov4/create_test.py ./

        !python create_train.py
        !python create_test.py

        both train.txt and test.txt should be in /data folder.

        train.txt = it has all the image paths for all of the training images.

    5.) Download yolov4 pre-trained weights.

        !wget https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v3_optimal/yolov4.conv.137

    6.) Train your custom yolov4 object detector.

        !./darknet detector train <path_to_obj.data> <path_to_custom_config> yolov4.conv.137 -dont_show -map 

    7.) Test your yolov4

        %cd cfg
        !sed -i 's/batch=64/batch=1/' yolov4-obj.cfg
        !sed -i 's/subdivisions=16/subdivisions=1/' yolov4-obj.cfg
        %cd ..

        !./darknet detector test data/obj.data cfg/yolov4-obj.cfg /mydrive/yolov4/backup/yolov4-obj_last.weights 
        /mydrive/images/car2.jpg -thresh 0.3
