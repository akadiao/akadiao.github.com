## 一.图像风格化


### 1.风格化


```markdown
PImage img;
int grid = 13 ; //line

void setup()
{
  size(915,1008); //
  img = loadImage("duck.jpg");  //
  noLoop();
  colorMode(HSB,255);
   background(0,0,255);
}

void draw()
{
  color c;
  fill(0);
  noStroke();
  ellipseMode(CENTER);
  for ( int y =0;y<height; y+= grid)
    for(int x = 0;x<width;x++)
    {
      c = img.get(int(map(x,0,width,0,img.width)),int(map(y,0,height,0,img.height)));
      ellipse(x,y+grid/2,grid*(1-brightness(c)/255),grid*(1-brightness(c)/255));
    }   
  save("pang.jpg");
}


```
![Image](https://github.com/akadiao/akadiao.github.com/blob/main/duck.jpg)

![Image](https://github.com/akadiao/akadiao.github.com/blob/main/pang.jpg)


















