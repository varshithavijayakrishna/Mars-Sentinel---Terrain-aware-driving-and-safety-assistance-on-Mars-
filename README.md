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
// FOR VIDEO INPUT
% Load the video file (can also use a webcam)
videoFile = "your_video_file";  % Specify your video path
vidObj = VideoReader(videoFile);

% Initialize the detector (YOLOv2 as an example)
detector = yolov2ObjectDetector('darknet19-coco');  % Using YOLO detector

% Create a figure for displaying the video
figure;

% Loop through the video frames to process each frame
while hasFrame(vidObj)
    I = readFrame(vidObj);  % Read the next frame
    
    % Detect objects in the current frame
    [bboxes, scores, labels] = detect(detector, I);
    
    % Display the frame
    I_resized = I;
    
    % Check if any objects were detected
    if ~isempty(bboxes)
        % Loop through the bounding boxes and labels to display them
        for i = 1:size(bboxes, 1)
            % Draw the bounding box
            I_resized = insertShape(I_resized, 'Rectangle', bboxes(i, :), 'Color', 'green', 'LineWidth', 3);
            
            % Add the label with confidence score next to the bounding box
            labelText = sprintf('%s: %.2f', char(labels(i)), scores(i));
            I_resized = insertText(I_resized, bboxes(i, 1:2), labelText, 'TextColor', 'white', 'FontSize', 12);
        end
    end
    
    % Display the image
    imshow(I_resized);
    title('Tracking and Detecting Objects');
    drawnow;  % Update the figure with new frame

end
