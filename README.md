# spark-image-processing

In order to eliminate human error when processing the image data, an algorithm was made in order to process all the photos taken. Using the algorithm, spark density and spark length were calculated and averaged across 106 photos and 11 different material types.

After acquisition, the photos were brought into an image processing software where they were converted to cropped images of a 10 inch square. After this bulk operation, every photo was vetted to ensure it was a usable sample for analysis with the algorithm. Photos with little or no sparks, incorrect alignment, and erroneous spark distributions caused by inconsistent pressure were removed from the data set. After the vetting process, the photos were loaded into MatLab where a gray scale conversion was performed. After conversion, a proportional brightness conversion was applied which prepared them for segmentation. The first step in the segmentation process was a simple threshold filter where luminance values were considered and all values below the threshold were crushed to black and all values above where pushed to white. After this first step, a morphological filter was applied to reduce noise. Below are images from each step of the process. 

In order to calculate spark density and spark length, the binarized images were examined and the area of white pixels was calculated and divided by the total area of the image. This could then be averaged across all data points in each sample and a single value could be calculated that represents the spark density for each sample. To calculate the spark length, the images were again examined and the major axis length of every region of white pixels was taken and averaged together per photo. This was then fed into a denoise algorithm which removed erroneous values and then finally averaged across all data points in a sample.




## Setting up Data 

    typeDir = dir("analysis/queue/")
    sparkDensityData = 'sparkDensityDatatest.xlsx'

## Importing Masks
Different masks are used for different scenarios as described in the title of the mask.

    import sparkMask.*
    import sparkMaskMorph.*
    import sparkMaskMorphBright.*
    import sparkMaskNorm.*
    import sparkMaskRGB.*

## Creating Spread Sheet Layout

    sparkMatrix = {'sparkArea', 'totalArea', 'sparkDensity','image','sparkLengthMean';}

## Beginning Iteration Over Folder Structure

    for j = 13: length(typeDir)
        
        fileFolder = fullfile('Analysis','queue',typeDir(j).name)
        dirOutput = dir(fullfile(fileFolder,'ENGR 140B Lab 1*.jpg'));
        addpath(fileFolder)
        fileNames = {dirOutput.name}'
        tempSpark = []

        for i = 1: length(fileNames)
    %     for i = 18: 18

## Import and Convert Image to Gray Scale

        
            imgSrc = imread(fileNames{i})
            imgSrcGray = imadjust(rgb2gray(imgSrc))
            imshow(imgSrcGray)
            rimg = imgSrc(:,:,1)
            gimg = imgSrc(:,:,2)
            bimg = imgSrc(:,:,3)
            
## Brightness Correction Basses on Averege Brightness of Image
            
            imgBrightness = mean(mean(imgSrcGray))
            imgBrightnessOffest = (40 - imgBrightness)*1.4
            workingImg = uint8(imgSrcGray + imgBrightnessOffest)
        %     imshow(workingImg)
        
            
            rimg = imadjust(rimg)
            gimg = imadjust(gimg)
            bimg = imadjust(bimg)
        
## Apply One of the Imported Masks

            imgSum = sparkMaskMorphBright(workingImg)
        
        %     imgSum = sparkMaskMorph(rimg)&sparkMaskMorph(gimg)&sparkMaskMorph(bimg)    
            
            subplot(2,2,1), imshow(workingImg)
            subplot(2,2,2), imshow(gimg)
            subplot(2,2,3), imshow(bimg)
            subplot(2,2,4), imshow(imgSum)
            
            
## Process Image Data and Write Image to File

        %     imshow(bin)
            imwrite(imgSum,strcat('BW - ',fileNames{i}),"jpeg")
            imgInfo = regionprops(imgSum,'MajorAxisLength','Area')
        
            sparkLength = [imgInfo.MajorAxisLength;]
            indicies = find(sparkLength<20)
            sparkLength(indicies) = []
            sparkLenghtMean = mean(sparkLength)/120
        
    %         imshow(imgSum)
            
        
            sparkArea = sum([imgInfo(:).Area])
            totalArea = numel(imgSum)
            
            sparkDensity = sparkArea / totalArea
        
            sparkInfo = {sparkArea, totalArea, sparkDensity,fileNames{i}, sparkLenghtMean;}
            tempSpark = [tempSpark;sparkInfo]
            sparkMatrix = [sparkMatrix;sparkInfo]
        end
        writecell(tempSpark,sparkDensityData,'Sheet',typeDir(j).name,'Range','A1' )
    end
    writecell(sparkMatrix,sparkDensityData,'Sheet','all','Range','A1' )