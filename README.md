# Mars-Sentinel---Terrain-aware-driving-and-safety-assistance-on-Mars
function marsDetectionGUIWithButtonsForVideo()
    % Create VideoReader object
    videoFile = "your_video_file"; % Specify your video path
    videoObj = VideoReader(videoFile);
    
    % Initialize the detector (YOLOv2)
    detector = yolov2ObjectDetector('darknet19-coco'); % Using YOLO detector

    % Create a figure for the GUI
    f = figure('Name', 'Mars Detection Video GUI', 'Position', [100, 100, 400, 300]);

    % Button to start processing the video
    uicontrol('Style', 'pushbutton', ...
        'String', 'Start Video', ...
        'FontSize', 12, ...
        'Position', [50, 200, 300, 40], ...
        'Callback', @(~, ~) startVideoProcessing(videoObj, detector));

    % Button for additional functionalities (if needed)
    uicontrol('Style', 'pushbutton', ...
        'String', 'Show Warnings', ...
        'FontSize', 12, ...
        'Position', [50, 20, 300, 40], ...
        'Callback', @(~, ~) disp('No warnings for now.')); % Placeholder
end

% Helper function to start video processing
function startVideoProcessing(videoObj, detector)
    while hasFrame(videoObj)
        % Read the next frame from the video
        frame = readFrame(videoObj);

        % Resize frame for YOLO detector
        resizedFrame = imresize(frame, [416, 416]); % YOLOv2 requires 416x416
        
        % Detect objects in the resized frame
        [bboxes, scores, labels] = detect(detector, resizedFrame);

        % Process detections
        if ~isempty(bboxes)
            % Rescale bounding boxes to original frame size
            scaleFactorX = size(frame, 2) / 416;
            scaleFactorY = size(frame, 1) / 416;
            bboxes(:, [1, 3]) = bboxes(:, [1, 3]) * scaleFactorX;
            bboxes(:, [2, 4]) = bboxes(:, [2, 4]) * scaleFactorY;

            % Annotate the frame
            annotatedFrame = insertObjectAnnotation(frame, 'rectangle', bboxes, labels);

            % Display detection results
            imshow(annotatedFrame);
            title('Mars Detection Video - Objects Detected');
        else
            % Display original frame if no detections
            imshow(frame);
            title('No Objects Detected');
        end

        % Debugging outputs
        disp("Bounding boxes:");
        disp(bboxes);
        disp("Scores:");
        disp(scores);
        disp("Labels:");
        disp(labels);

        drawnow; % Refresh the figure window
        pause(0.1); % Adjust frame rate
    end
end
