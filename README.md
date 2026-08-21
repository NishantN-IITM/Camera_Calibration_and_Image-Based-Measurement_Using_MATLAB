# Image-analysis - MATLAB Pipeline
MATLAB-based image analysis pipeline for automated extraction, processing, and quantitative characterization of experimental data from peeling and Interfacial fracture studies.

<details>
<summary><b>MATLAB Code Implementation</b></summary>

```matlab

clc
clear

inputFolder = 'C:\Users\user\OneDrive\Desktop\IMAGE PROCESSING\PRACTICE TEST\CONCAVE TESTS\TEST 1\GH016130_undistorted_frames';   % Folder containing all frames

outputFolder = fullfile(inputFolder,'Processed_Figures');
if ~exist(outputFolder,'dir')
    mkdir(outputFolder);
end


lastFrame = 31702;                  % Total number of frames

I = imread(fullfile(inputFolder, 'frame_00001.jpg'));

% Display (optional)
figure(1)
imshow(I)
title('Select two reference points')

[x_ref,y_ref] = ginput(2);
    
pixel_dist = sqrt( ...
(x_ref(2)-x_ref(1))^2 + ...
(y_ref(2)-y_ref(1))^2 );
    
real_dist_mm = 100;
    
mm_per_pixel = real_dist_mm / pixel_dist;

results= [];
angles = [];

```
Also taking a separate video of Checkerboard patterns to calibrate it later to know about the World Coordinates in later calculations...
<img width="1920" height="1080" alt="frame_1920" src="https://github.com/user-attachments/assets/55d5b04b-718b-4756-8841-ca874e8c80b4" />

```MATLAB
    
% Making a Loop for Frames

for frameNumber = 1:1800:lastFrame

    filename = fullfile(inputFolder, sprintf('frame_%05d.jpg', frameNumber));

    if ~isfile(filename)
        continue
    end

      I = imread(filename);

    
    % ADJUST THE INTENSITY OF THE RED COLOUR 
    R = I(:,:,1);
    G = I(:,:,2);
    B = I(:,:,3);

    HSV = rgb2hsv(I);
    
    H = HSV(:,:,1);
    S = HSV(:,:,2);
    V = HSV(:,:,3);

    redMask = ((H < 0.05 | H > 0.95) & ...
                S > 0.4 & ...
                V > 0.4);
    
    imshow(redMask)
    
    redMask = bwareaopen(redMask,20);
    
    redMask = imfill(redMask,'holes');
    
    imshow(redMask)
    
    stats = regionprops(redMask,...
        'Centroid',...
        'Area',...
        'EquivDiameter');

    if isempty(stats)
        continue
    end
    
    centroids = cat(1,stats.Centroid);
    
    disp(centroids)   % Marking the centroids of RED Circles

    x_mm = centroids(:,1) * mm_per_pixel;
    y_mm = centroids(:,2) * mm_per_pixel;

     results = [results;
               frameNumber x_mm(1) y_mm(1)];

% Identifying the Red Coloured circle feature from the Image & marking centroids on it which was basically placed manually on the frontface of the assembly..

```
Marking Centroid on the centres of the Red circles...
<img width="1305" height="736" alt="Figure_2" src="https://github.com/user-attachments/assets/44ebb454-3aa7-412f-8330-3a62d2cb867f" />
Converting the same Image into Black & White...
<img width="2606" height="1468" alt="Angle_vs_Time" src="https://github.com/user-attachments/assets/7e6f0951-23a9-4c1a-b3c6-759c06cf2a8b" />

```MATLAB

%% Display

    fig = figure('Visible', 'off');

    imshow(I)
    hold on

    plot(centroids(:,1),centroids(:,2),...
        'r+','MarkerSize',15,'LineWidth',2)

    tableText = {};



```
Displaying one of the frame before plotting any Vector on it

<img width="1920" height="1080" alt="frame_21623" src="https://github.com/user-attachments/assets/66e0cc02-d2bd-4584-9702-0f1eca37265f" />

```MATLAB

% Number the points
for k = 1:size(centroids,1)

    text(centroids(k,1)+8, centroids(k,2)-8,...
        sprintf('P%d',k),...
        'Color','yellow',...
        'FontSize',10,...
        'FontWeight','bold',...
        'BackgroundColor','black');

end

tableText = sprintf('Point      X (px)      Y (px)\n');
tableText = [tableText sprintf('----------------------------\n')];

for k = 1:size(centroids,1)
    tableText = [tableText,...
        sprintf('P%-2d   %8.1f   %8.1f\n',...
        k,centroids(k,1),centroids(k,2))];
end



% Connect P5-P3
line([centroids(5,1) centroids(3,1)], ...
     [centroids(5,2) centroids(3,2)], ...
     'Color','g','LineWidth',1, 'Linestyle','--');

% Connect P6-P4
line([centroids(6,1) centroids(4,1)], ...
     [centroids(6,2) centroids(4,2)], ...
     'Color','g','LineWidth',1, 'Linestyle','--');



% Vector P5 -> P3
v1 = centroids(3,:) - centroids(5,:);

% Vector P6 -> P4
v2 = centroids(4,:) - centroids(6,:);

theta = acosd( dot(v1,v2) / (norm(v1)*norm(v2)) );
angles = [angles;
          frameNumber theta];

% Coordinates table
annotation('textbox',...
    [0.72 0.60 0.25 0.30],...
    'String',tableText,...
    'FitBoxToText','on',...
    'FontName','Courier New',...
    'FontSize',10,...
    'BackgroundColor','black',...
    'Color','yellow');
    
    exportgraphics(fig,...
    fullfile(outputFolder,...
    sprintf('Figure_%05d.png',frameNumber)),...
    'ContentType','vector');

    title(sprintf('Frame %05d',frameNumber))
    drawnow

    close(fig)

end
% Displaying the First & the Last Processed frames extracted after Image Calibration 

```
First Frame,
<img width="1390" height="758" alt="Figure_03601" src="https://github.com/user-attachments/assets/ad4670a2-a4df-4024-bd06-8c904d8efed2" />
Last Frame,
<img width="1390" height="758" alt="Figure_30601" src="https://github.com/user-attachments/assets/39c8a96c-3176-4ccb-a039-7d1c7c778a4f" />

```MATLAB
%% Plot Angle VS Time

sampleInterval = 1800;   % Every 1800th frame processed
fps = 30;                % Video frame rate

time = (0:size(angles,1)-1) * sampleInterval / fps;

figure

plot(time, angles(:,2), '-o', ...
    'LineWidth',2, ...
    'MarkerSize',6)

xlabel('Time (s)')
ylabel('Included Angle (degrees)')
title('Angle vs Time')

grid on

%% Plot Angle VS Frame Number

figure

plot(angles(:,1), angles(:,2), '-o', ...
    'LineWidth',2, ...
    'MarkerSize',6)

xlabel('Frame Number')
ylabel('Included Angle (degrees)')
title('Angle vs Frame Number')

grid on

%% Create Table

angleTable = table( ...
    angles(:,1), ...
    time', ...
    angles(:,2), ...
    'VariableNames', ...
    {'Frame_Number','Time_s','Angle_deg'});

disp(angleTable)

writetable(angleTable,'Angle_Results.xlsx');

```
Plotting the trend of angle between these two vectors from first frame to the last frame (31702)
<img width="2417" height="1332" alt="Variation of angles wrt Frames" src="https://github.com/user-attachments/assets/77b2a997-879e-488b-9856-e03f3b42481f" />

</details>





