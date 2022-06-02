# CSACF
Title: Color-Saliency-Aware Correlation Filters with Approximate Affine Transform for Visual Tracking.
>
The paper has been accepted for publication in The Visual Computer.
>
![Fig1](https://github.com/lv346308962/CSACF/blob/03a8452c99fdb6c984b4790e37b87b3f74a3d9ee/imgs/frame.png)
## Updates
> The results of the experiment have been uploaded.
> 
> At present, only the core code has been uploaded. To quickly implement the visual tracking with approximate affine transform, you can replace tracking.m of LDES tracker (https://github.com/ihpdep/LDES), set p.dc = 1 and p.mbs = 0. 
> 
> The complete code will be uploaded after finishing. Thanks!
## Acknowledgements
We thank for Dr. Yang Li and Dr. Jianming Zhang for their basic work before. In this work, we have borrowed the similarity scale estimation modules from the LDES tracker and the saliency detection modules from MB+ mthod (https://github.com/jimmie33/MBS).
## Example
### Red: CSACF ------------ Green: LDES ------------ Blue: ECO-HC ------------ Black BACF ------------ Pink: Staple
<img src="https://github.com/lv346308962/CSACF/blob/a8205a3d22d421202daadd9824a4196d6211dbad/imgs/test1.gif" width="200px"/>

