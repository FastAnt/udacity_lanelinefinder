# **Finding Lane Lines on the Road** 

## Artem Melnytskyi, 2020

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./res/original.jpeg "Grayscale"
[image2]:./res/blur_gray.jpeg
[image3]:./res/edges.jpeg
[image4]:./res/end.jpeg
[image5]:./res/hls_image.jpeg
[image6]:./res/lines.jpeg
[image7]:./res/mask.jpeg
[image8]:./res/masked_edges.jpeg
[image9]:./res/original.jpeg
[image10]:./res/white_mask.jpeg
[image11]:./res/yellow_mask.jpeg
[image12]:./res/1.png
[image13]:./res/2.png


---

### Reflection

###  Pipeline.

####1. Proprocess image
![alt text][image1]
#####1.1 extract color, and yellow colors from image 
    
For hightlite lanes on road, need to understand that for white and for yellow lanes extraction will be better to use different color schemes 
![alt text][image5]
White  - RGB
Yellow - HLS
    `hls_image = cv2.cvtColor(original_image, cv2.COLOR_RGB2HLS) `   # lets convert to HLS
    
    `yellow_mask = cv2.inRange(hls_image, (10, 0, 100), (40,255,255))` # take yellow mask from HLS
    
    `white_mask = cv2.inRange(original_image, (180, 180, 180), (255,255,255))` #take white mask from RGB
    
    `white_mask = white_mask + cv2.inRange(hls_image, (0, 100, 100), (40,255,255))` # update white mask from HLS
    

YELLOW ![alt text][image11]                       { WHITE ![alt text][image10]}

###### 1.2 combine masks ( from RGB and HLS )
    `mask_image =  white_mask + yellow_mask `    # lets take two masks (from hls - yellow , from rgb - white)
    
    `hls_image[mask_image>0] = [0,0,0]`

###### 1.3 Convert to gray scheme.
`` gray = cv2.cvtColor(original_image, cv2.COLOR_RGB2GRAY)
    blur_gray = cv2.GaussianBlur(gray, (kernel_size, kernel_size), 0)``
![alt text][image2]

######1.4 Add blur for removing noise, defects on road and ect
######1.5 Use Canny edges 
`` edges = cv2.Canny(blur_gray, low_threshold, high_threshold)
    mask = np.zeros_like(edges)``
    
![alt text][image3]

######1.6 Set ROI 
``cv2.fillPoly(mask, get_vertices(original_image), ignore_mask_color)``
    
  ![alt text][image7]  
######1.7 Bitwise mask and cany
``masked_edges = cv2.bitwise_and(edges, mask)``
  ![alt text][image8]  


#### 2.  Process HoughLines, extract parameters ,filter parameters, build lanes.
Theory :
![alt text][image12]
![alt text][image13]
Implementation: 
``slope = (x2-x1)/(y2-y1)
  intercept =  y2  - (x2 / slope)``
  
        `` if slope > 0 :

            l_slope_lst.append(slope * leng)
            
            l_intercept_lst.append(intercept * leng)
            
            total_l_len = total_l_len + leng
            
        else:
        
            r_slope_lst.append(slope * leng)
            
            r_intercept_lst.append(intercept * leng)
            
            total_r_len = total_r_len + leng``
            
  Also in code you can find logic of weights of lines. Weight of line in calulation ~ length of line.
  
 ###### 2.3 Filter parameters
 For data balancing can be used simple kind of Bayesian  filtering.
 Detail implementation in class Filter.
####### 2.4 Build lanes.
``cv2.line(canva, (int(l_x[0]), int(l_y[0])), (int(l_x[1]), int(l_y[1])), (255, 0, 0), 25)``

``cv2.line(canva, (int(r_x[0]), int(r_y[0])), (int(r_x[1]), int(r_y[1])), (255, 0, 0), 25)``

### 2 Potential problems
1 .In cases where lines will have abnormal shape (not parallel and have negative degree in ego-centric system)### 3. Suggest possible improvements to your pipeline
2. Big dependancy on camera position in car.
### Improovment
1. Some kind lightweight NN
2. Kalman filters
3. Line from polynome, that will get as approximation of points from HoughLines

