## 一.图像风格化


### 1.条纹风格化


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

![Image](https://github.com/akadiao/akadiao.github.com/blob/main/pang.jpg?raw=true)




### 2.旋转图形


```markdown

float initSize = 220;
float scaleFactor = 0.986;

void setup() {
  size(600, 600);
  background(255);
  noFill();
  strokeWeight(1.5);
  stroke(#006211);
  noLoop();
}

void draw() {
  translate(width/2, height/2);
  rotate(-PI/2);
  for (int i = 0; i<120; i++) {
    float size = initSize*pow(scaleFactor, i);
    rect(0, 0, size, size);
    rotate(PI/24);
    scaleFactor*=0.99995;
  }
}


```
![image](https://github.com/akadiao/akadiao.github.com/blob/main/Rotating%20cube.jpg)





### 2.苹果logo


```markdown
PImage img;
ArrayList <Circle> circles = new ArrayList <Circle> ();
int shiftx, shifty;
void setup(){
  fullScreen();
  //noLoop();
  colorMode(HSB);
  background(255);
  smooth();
  noStroke();
  img = loadImage("Apple_logo_black.jpg");
  shiftx = width/2-img.width/2;
  shifty = height/2-img.height/2;
  img.loadPixels();
  //image(img,shiftx,shifty);
}
void draw(){
  //img.loadPixels();
  background(255);
  //image(img,shiftx,shifty);
  if(circles.size()>0){
    for(Circle c : circles){
      c.display();
    }
  }
  float xn, yn;
  while(true){
    xn = (randomGaussian()*200)+mouseX;
    yn = (randomGaussian()*200)+mouseY;
    int xns,yns;
    xns = (int) (xn-shiftx);
    yns = (int) (yn-shifty);
    if (xns<0 || xns>=img.width || yns<0 || yns>=img.height) break;
    int loc = xns + yns*img.width;
    float b=brightness(img.pixels[loc]);
    if(b>50){
      break;
    }
  }
  boolean sign = true;
  for(Circle c:circles){
    if(dist(xn,yn,c.x,c.y)<c.r+2){
      sign = false;
      break;
    }
  }
  if(sign){     
    Circle cir = new Circle(xn,yn);
    cir.grow(randomGaussian()*5+10);
    circles.add(cir);
  }
}



class Circle{
  float x,y,r;
  color c;
  Circle(float xin, float yin){
    x = xin;
    y = yin;
    c = color(random(255),255,255);
  }
  void display(){
    noStroke();
    fill(c);
    ellipse(x,y,2*r,2*r);
  }
  void grow(float rmax){
    for(float ri=2; ri<rmax; ri+=1){
      r = ri;
      boolean sign1 = false;
      for(Circle c : circles){
        if(dist(x,y,c.x,c.y) <= c.r+r){
          sign1 = true;
          break;
        }
      }
      if(sign1){
        break;
      }
      boolean sign2 = false;
      for(int i=0; i<360; i++){
        float rad = radians(i);
        float xa = x+cos(rad)*r;
        float ya = y+sin(rad)*r;
        int xs = (int) (xa-shiftx);
        int ys = (int) (ya-shifty);
        if(xs<0 || xs>=img.width || ys<0 || ys>=img.height) break;
          int loc = (int) (xs+ys*img.width);
          float b = brightness(img.pixels[loc]);
          if(b<50){
            sign2 = true;
            break;
          }
        }
      if(sign2){
        break;
      }
    }
  }
}
void mousePressed(){
  save("apple2.png");
}


```
![image](https://github.com/akadiao/akadiao.github.com/blob/main/Apple_logo_black.jpg)
![image](https://github.com/akadiao/akadiao.github.com/blob/main/AppleLogo.jpg)


