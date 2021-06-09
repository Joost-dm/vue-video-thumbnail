# vue-video-thumbnail
A component that creates snapshots from video. Only video src is required.
A lot of props for customize, slot for custom snapshot rendering.
Can create multiple snapshots.  
I hope it will be helpful for you ;)


Demo: https://joost-dm.github.io/vue-video-thumbnail/
## Project setup
```
npm install vue-video-thumbnail
```

## Usage
```
import VueVideoThumbnail from 'vue-video-thumbnail'
    ...
    
components: { VueVideoThumbnail },
    ...
```

```
    <VueVideoThumbnail
      video-src="your source"
      show-play-button
      :snapshot-at-duration-percent="70"
      :width="500"
      :height="300"
      :chunks-quantity="10"
      @snapshotCreated=" ***place for handler of single snapshot*** "
      @snapshotsArrayCreated=" ***place for handler of array of snapshots*** "
    >
      <template #snapshot="{snapshot}">
        <img
          v-if="snapshot"
          :src="snapshot"
          alt="snapshot"
        >
      </template>
    </VueVideoThumbnail>
```
## Slots
##### #snapshot - slot for custom snapshot rendering
```
    <template #snapshot="{snapshot}"> 
        <img
          v-if="snapshot"
          :src="snapshot"
          alt="snapshot"
        >
    </template>
```

## Events
 ##### @snapshotCreated
    Returns a single snapshot when its ready

##### @snapshotsArrayCreated
    Returns an array of snapshots when they are ready.
    Don't forget to send percentagesArray or chunksQuantity props.
## Props

##### videoSrc:
###### video source 
    type: String
    required
    
##### renderThumbnail:
###### if true component renders thumbnail
    type: Boolean
    default: true

##### width:
###### width of snapshot in px
    Number
    default: 250
      
##### height:
###### height of snapshot in px
    type: Number
    default: 250
    
##### snapshotAtDurationPercent:
###### create a snapshot at percent of the video duration.
    type: Number
    default: 50

##### snapshotAtTime: 
###### create a snapshot at fixed time, have a less priority than snapshotAtDurationPercent prop.
    type: Number
    default: 2
    
##### snapshotScaleType:
###### scale type ov snapshot. Avialable values are: 'contain' and 'cover'
    type: String
    default: 'contain'
    
##### backgroundFillColor: 
###### background snapshot color at 'contain' scale-type
    type: String
    default: '#000000'
    
##### snapshotQuality:
###### A Number between 0 and 1 indicating the image quality to use for image formats that use lossy compression such as image/jpeg and image/webp.
    type: Number
    default: 0.8
    
##### snapshotFormat: 
###### snapshot format. Have the same restrictions as HTMLCanvasElement.toDataURL() method https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL
    type: String
    default: 'image/jpeg'
    
##### cors: 
###### if false add the 'crossOrigin': 'Anonymous' attribute to video el.
    type: Boolean,
    default: false,
    
##### percentagesArray:
###### you can send an array with percentages, to recive snapshots at these percentages timestamps.
    type: Array of numbers
    default: null
    example: [10, 25, 95]
        
##### chunksQuantity:
###### number of snapshots, that will be created, evenly distributing through the video duration.
    type: Number
    default: null
     
##### once:
###### if set to true, the component destroys video and canvas elements to clear the DOM after creating first snapshot or snapshotsArray, if its required, so the creation of new snapshots become impossible after that.
    type: Boolean
    default: false

##### showPlayButton:
###### if set to true, shows the "play" button at the snapshot center.
    type: Boolean
    default: false


