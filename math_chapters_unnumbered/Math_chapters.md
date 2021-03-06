# That Math Chapter: From 1D to 4D 

## Intro: How Artists Approach Math
** // TODO: Write intro **

…

## About this Chapter
This chapter will be divided into _'numbers of D's'_ : we'll start from one dimension, and slowly explore the possibilities of using scale and change in different dimensions. Depending on how you choose to read it, this section contains hundreds of Mathematicians' lifetime research, or in other words, several classes of college math, so it's worth bookmarking.

When bringing math to innocent readers, most programming books will try to explain the idea, not necessarily the exact implementation. This book is no different. This chapter contains detailed breakdowns of concepts, but if you want to find out what's going on under the hood, there's no alternative to reading the source code - in fact, since all the math here is only a few lines long - it's actually _encouraged_ to have a look at the source.

## One Dimension: Using Change
** // TODO: Write intro **
### Interpolation
#### Linear Interpolation: The `ofLerp`
```float ofLerp(float start, float stop, float amt)```

As xkcd once put it, if you've seen a number larger than 7, you're not doing real math.

Those of you that have already done a little time-based or space-based work have probably noticed that you can often describe elements of your work as sitting on a line between two known points. A frame on a timeline is at a known location between 00:00:00 and the runtime of the film, a scrollbar is pointing to a known location between the beginning and the end of a page. That's exactly what `lerp` does.

With the `lerp` function, you can take any two quantities, in our case `start` and `stop`, and find any point between them, using amounts (`amt`) between 0 and 1. To be verbose: 
 $$\text{lerp}\left(a,b,t\right) = t\cdot b+\left(1-t\right)\cdot a$$ 

##### Note: What does _linear_ really mean?
Engineers, Programmers and English Speakers like to think of _linear_ as _anything you can put on a line_. Mathematicians, having to deal with all the conceptual mess the former group of people creates, define it _anything you can put on a line **that begins at (0,0)**_. There's  good reasoning behind that, which we will see in the discussion about Linear Algebra. In the meantime, think of it this way: if our transformation is taking a line that has a value 0 at the point 0 and returning a line with the same property, (thus in the form $$$f\left(x\right)=ax$$$), It's _linear_. If it returns a value different from 0 at $$$x=0$$$ (in the form $$$f\left(x\right)=ax + b$$$), it's _affine_. 

##### Exercise: Save NASA's Mars Lander 
In 1999, an masterpiece of engineering was making its final approach to Mars. All instruments were showing that the approach distance matched the speed, and that it's just about to get there and do some science. But instead, it did something rather rude: it crashed into the red planet. An investigation made later by NASA revealed that while designing the lander, one team worked with their test equipment set to _centimetres_, while the other had theirs set to _inches_. **By the way, this is all true.**

Help the NASA teams work together: write a function that converts centimetres to inches. For reference, $$$1\_{\text{in}} = 2.54\_{\text{cm}}$$$. Test your result against three different real-world values.

**Think:**

1. Why can we use `lerp` outside the range of 0 and 1?
2. What would it take to write a function that converts inches into centimetres?

#### Affine Mapping: The `ofMap`
`float ofMap(float value, float inputMin, float inputMax, float outputMin, float outputMax, bool clamp = false)`


In the last discussion, we saw how by using `lerp`, any value between two points can be _linearly_ addressed as a value between 0 and 1. That's very convenient, and therefore the reason we build most of our arbitrary numerical ranges (`ofFloatColor`, for example) in the domain of 0 and 1. 
However, when dealing with real world problems, programmers run into domains of values that they wish to map to other ranges of values, neither of which are confined to 0 and 1. For example, someone trying to convert the temperature in Celsius to Fahrenheit won't be able to use  Surely, the way of doing that must involve a `lerp`, but it needs a little help:

If we want to use the `lerp` function, we're aiming to get it to the range between 0 and 1. We can do that by knocking `inputMin` off the input `value` so that it starts at 0, then dividing by the size of the domain: $$x=\frac{\text{value}-\text{inputMin}}{\text{inputMax}-\text{inputMin}}$$
Now that we've tamed the input domain to be between 0 and 1, we do the exact opposite to the output: `ofMap(value, inputMin, inputMax, outputMin, outputMax)` $$$=\frac{\text{value}-\text{inputMin}}{\text{inputMax}-\text{inputMin}}\cdot\left(\text{outputMax}-\text{outputMin}\right)+\text{outputMin}$$$

#### Range Utilities
##### Clamping
You'll notice that the previous explanation is missing the `clamp` parameter. This may not matter to us if we're using the `ofMap` function in the range that we defined, but suppose we select a `value` smaller than `inputMin`: would it be ok if the result was also smaller than `outputMin`? If our program is telling an elevator which floor to go to, that might be a problem. That's why we add `true` to the tail of this function whenever we need to be careful.

Just in case, oF offers another specific clamp function: 
```
float ofClamp(float value, float min, float max)
```
##### Range Checking
Two important functions we unjustly left out of this episode;

```
bool ofInRange(float t, float min, float max);
```
Tells you whether a number `t` is between `min` and `max`. 

```
float ofSign(float n);
```
Returns the sign of a number, as `-1.0` or `1.0`. Simple, eh?

### Beyond Linear: Changing Change

So far we've discussed change that is bound to a line. But in Real Life™ there's more than just straight lines. 

In this discussion, we're about to see how we can describe higher orders of complexity, via a cunning use of `lerp`s. Keep in mind that some of the code here is conceptual, not necessarily efficient.

#### Quadratic and Cubic Change Rates
Consider this function:

```
float foo (float t){
	float a1 = ofLerp(t, 5, 8);
	float a2 = ofLerp(t, 2, 9);
	float b = ofLerp(t, a1, a2);
	return b;
	}
```
This function used a defined range and a parameter to create `a1`, then used another defined range with the _same_ parameter to create `a2`. Their result looks surprising:

**//TODO: Draw quadratic eqn**

We've done something remarkable here. We used the way one parameter changes on two fixed lines to control a third, totally mobile line, and draw one point on it at each point in time between 0 and 1. In Mathspeak, it looks like this:
 
$$
\text{lerp}\left(t,\text{lerp}\left(t,5,8\right),\text{lerp}\left(t,2,9\right)\right)\\\\
= \text{lerp}\left(t,8\cdot t+5\cdot\left(1-t\right),9\cdot t+2\cdot\left(1-t\right)\right)\\\\
= \left(9\cdot t+2\cdot\left(1-t\right)\right)\cdot t+\left(8\cdot t+5\cdot\left(1-t\right)\right)\cdot\left(1-t\right)\\\\
= \left(9t^{2}+2t-2t^{2}\right)+\left(8t+5-5t\right)-\left(8t^{2}+5t-5t^{2}\right)\\\\
= 4t^{2}+5
$$

Something interesting happened here. Without noticing, we introduced a second order of complexity, a _quadratic_ one. If you don't find this relationship remarkable, please give this book to someone who does.

**//TODO: Write code for this example**

The same manipulation can be applied for a third order:

```
float foo (float t){
	float a1 = ofLerp(t, 5, 8);
	float a2 = ofLerp(t, 2, 9);
	float a3 = ofLerp(t, 3, -11);
	float a4 = ofLerp(t, -2, 4);
	float b1 = ofLerp(t, a1, a2);
	float b2 = ofLerp(t, a3, a4);
	float c = ofLerp(t, b1, b2);
	return c;
	}
```
We'll skip the entire solution, and just reveal that the result will appear in the form of $$ax^{3} + bx^{2} + cx + d$$
See the pattern here? The highest exponent is the number of successive `ofLerp`s we applied, i.e. the number of successive times we changed using our parameter $$$t$$$.

**//TODO: Add thanks to Steven Wittens for the idea of lerping**

##### …And So On
The general notion in Uni level Calculus is that _you can do anything if you have enough of something_. So fittingly, there's a curious little idea in Mathematics which allows us, with enough of these nested control points, to approximate any curve segment we can imagine. In the original formulation of that idea (called a _Taylor Series_), we only reach a good approximation if the amount of degrees (successive `lerp`s we applied) is close to infinity.

In Computer Graphics, as you're about to see - 3 is close enough.

### Splines
What we've done in the previous chapter is really quite remarkable. We have built a series of control points for 
** //TODO: Write this **
### Tweening
** //TODO: Write this **
#### Example:
Make a ball bounce, an eye blink, and a door to slam from the wind.

## More Dimensions: Some Linear Algebra
Until now, we explored several ideas on how to change what's going on the number line. That's cool, but we want to know how to do graphics, and graphics has more than one dimension. Our ancient Mathematician ancestors (Just kidding, most important Mathematicians die before 30. Not kidding) also faced this problem when trying to address the space of shapes and structures, and invented some complex machinery to do so. The fancy name for this machinery is _Linear Algebra_, which is exactly what it sounds like: using algebraic operations (add and multiply, mostly), in order to control many lines. 

In this part you're going to learn many concepts in how to store and manipulate multidimensional information. You'll later be able to use that information to control realtime 3d graphics using OpenGL, and impress the opposite (or same) sex with your mastery of geometry.

### The Vector
You may have heard of vectors before when discussing directions or position, and after understanding that they can represent both, may have gotten a little confused. Here's The Truth About Vectors™: 
	
	A vector is just an array that stores multiple pieces of the same type information. 

Seriously, that's all it is. Quit hiding.

This simplicity is also their great power. Just like The number 5 can be used to describe five Kilometres, the result of subtracting 12 and 7, or the number of cookies in a jar - the same works with vectors.

It's up to the user of that mathematical object to choose what it is used as. The vector $$$v=\left(5,-3,1\right)$$$ can represent a point in space, a direction of a moving object, a force applied to your game character, or just three numbers. And just like with numbers, algebraic operations such as addition and multiplication may be applied to vectors. 

Oh, but there's a catch. You see, everyone was taught what $$$a + b$$$ means. In order to go on with vectors, we need to define that.

#### Vector Algebra
Generally speaking, when dealing with Algebra of numerical structures that aren't numbers, we need to pay close attention to the _type_ of things we're cooking together. In the case of vectors, we'll make a distinction between _per-component_ and _per-vector_ operations.

##### Scalar Multiplication
The product between a vector and a scalar is defined as: 
$$a\left(\begin{array}{c}
x\\\\
y\\\\
z
\end{array}\right)=\left(\begin{array}{c}
ax\\\\
ay\\\\
az
\end{array}\right)$$
That falls into the category of _per-vector_ operations, because the entire vector undergoes the same operation. Note that this operation is just a scaling.  

```
ofVec3f a(1,2,3);
cout << ofToString( a * 2 ) << endl; 
//prints (2,4,6)
```

##### Vector Addition
Adding vectors is pretty straightforward: it's a _per-component_ operation:

$$\left(\begin{array}{c}
x\_{1}\\\\
y\_{1}\\\\
z\_{1}
\end{array}\right)+\left(\begin{array}{c}
x\_{2}\\\\
y\_{2}\\\\
z\_{2}
\end{array}\right)=\left(\begin{array}{c}
x\_{1}+x\_{2}\\\\
y\_{1}+y\_{2}\\\\
z\_{1}+z\_{2}
\end{array}\right)$$ 


```
ofVec3f a(10,20,30);
ofVec3f b(4,5,6);
cout << ofToString( a + b ) << endl; 
//prints (14,25,36)
```
###### Example: `ofVec2f` as position
Vector addition serves many simple roles. In this example, we're trying to track our friend Gary as he makes his way home from a pub. Trouble is, Gary's a little drunk. He knows he lives south of the pub, so he ventures south; But since he can't walk straight, he might end up somewhere else.

```
/* in Gary.h: */
class Gary {
public:
	void setPosition(ofVec2f initialPosition){ position = initialPosition; }
	void step(ofVec2f direction){ position += direction; }
	ofVec2f getPosition(){ return position; }
	void draw(){
		//TODO: Fill this
	}
private:
	ofVec2f position;
}

/* in testApp.h: */
Gary gary;

/* in testApp.cpp: */
void testApp::setup(){
	ofVec2f garysStartingPoint( ofGetWidth() / 2., ofGetHeight() / 3. );
	gary.setPosition( garysStartingPoint );
}

void testApp::update(){
	if (gary.position.y < ofGetHeight * 2. / 3.){
		ofVec2f nextStep(ofRandom(-1.,1.),1.); //Take one step south, Gary
		gary.step();
	}
}

void testApp::draw(){ gary.draw(); }

```

###### Example: `ofVec2f` as velocity

##### Note: C++ Operator Overloading
Just like we had to define the meaning of a product of a scalar quantity and a vector, programming languages - working with abstract representations of mathematical objects, also need to have definitions of such an operation built in. C++ takes special care of these cases, using a feature called _Operator Overloading_: defining the `*` operation to accept a scalar quantity and a vector as left-had side and right-hand side arguments:

```
ofVec3f operator*( float f, const ofVec3f& vec ) {
    return ofVec3f( f*vec.x, f*vec.y, f*vec.z );
}
```

The same is defined, for example, between two instances of `ofVec3f`:

```
ofVec3f ofVec3f::operator+( const ofVec3f& pnt ) const {
	return ofVec3f( x+pnt.x, y+pnt.y, z+pnt.z );
}
```

naturally representing the idea of vector addition.

The basic arithmetic operations, `+`, `-`, `*`, `/`,`+=`, `-=`, `*=`, `/=`, exist for both combinations of `ofVec2f`, `ofVec3f` and `ofVec4f`s and between any vector object and a scalar quantity.

Some excellent examples of operator overloading done right exist in the source files for the `ofVec` types. It's encouraged to check them out.

**Warning: Overloading operators will make you go blind.** Programmers use operators without checking what they do, so bugs resulting from bad overloads take a long time to catch. If the expression `a + b` returns a reference instead of a copy, a `null` instead of a value, or doing a complex operation which may crash, you've entered a world of pain. Unless the operator can do one arithmetic thing and that alone, don't change operators. Go to Appendix III and sign a form saying you understand that.

##### Distance Between Points
```
float ofVec3f::distance( const ofVec3f& pnt) const
float ofVec3f::squareDistance( const ofVec3f& pnt ) const
float ofVec3f::length() const
float ofDist(float x1, float y1, float x2, float y2);
float ofDistSquared(float x1, float y1, float x2, float y2);
```
Let's start by a definition. You may remember the _Pythagorean Theorem_, stating that the length of a line between point $$$a$$$ and $$$b$$$ is:
$$\text{Distance}\left(\left(\begin{array}{c}
x\_{a}\\\\
y\_{a}
\end{array}\right),\left(\begin{array}{c}
x\_{b}\\\\
y\_{b}
\end{array}\right)\right)=\sqrt{\left(x\_{b}-x\_{a}\right)^{2}+\left(y\_{b}-y\_{a}\right)^{2}}$$

Here's the good news: It's the exact same definition in three dimensions! just add the $$$z$$$ term.
$$\text{Distance}\left(\left(\begin{array}{c}
x\_{a}\\\\
y\_{a}\\\\
z\_{a}
\end{array}\right),\left(\begin{array}{c}
x\_{b}\\\\
y\_{b}\\\\
z\_{b}
\end{array}\right)\right)=\sqrt{\left(x\_{b}-x\_{a}\right)^{2}+\left(y\_{b}-y\_{a}\right)^{2}+\left(z\_{b}-z\_{a}\right)^{2}}$$


Vector Length, then, can be naturally defined as the distance between the vector and the point $$$\left(0,0,0\right)$$$:
$$\text{Length}\left(\begin{array}{c}
x\\\\
y\\\\
z
\end{array}\right)=\sqrt{x^{2} + y^{2} + z^{2}}$$

And that's exactly what using `.length()` as a property of any `ofVec` will give you.

##### Vector Products: There's More Than One
So you're multiplying two numbers. Simple, right? Five million and seven times three equals something you know. Even if you need a calculator for the result, you still know _it's a number_ that's not the case with vectors. If we just want to resize vectors (the way we do with numbers), we multiply a vector by a scalar and it grows. But what does it mean, geometrically, to multiply by a vector?

If we were to follow the _per-component_ convention that we created, we would get an operation like this:
```
cout << ofToString(ofVec3f(1,2,3) * ofVec3f(1,2,3)) << endl;
//prints (1,4,9)
```
It's also known as the _Hadamard product_. It's intuitive, but not particularly useful. One case it is useful for is if we want to scale something individually in every dimension.

In the next section we describe something more helpful.

##### The Dot Product
```
float ofVec3f::dot( const ofVec3f& vec )
```
The dot product of two vectors has a definition that's not too clear at first. On the one hand, the operation can be defined as $$$v\_{a}\bullet v\_{b}=x_{a}\cdot x\_{b}+y\_{a}\cdot y\_{b}+z\_{a}\cdot z\_{b}$$$, which is really easy to implement, on the other hand, it can also bet defined as $$$v\_{a}\bullet v\_{b}=\left\Vert v\_{a}\right\Vert \cdot\left\Vert v\_{b}\right\Vert \cdot\cos\theta$$$, where $$$\theta$$$ is the angle between the two vectors.

For reasons you'll learn soon, it's a rather surprising coincidence.

**//TODO: Finish this**


##### Example: Dot product for playing billiards in 2D

### The Matrix™ 
In the computer world, a program needs the two things to function: Algorithms and Data Structures (it also needs I/O, but we're talking Turings, not Perlins). In the 3D Maths world it's exactly the same: we call our data structures 'vectors' and our algorithms are operations. 

At the core of the heavy machinery built to control 3d space, a matrix is just a data structure, like a vector. However, the 'algorithms' applied to this data structure (operations, in Mathland) make it an extremely powerful one. All of the _affine_ operations we care about in 3D can be described in the form of a matrix: translation, rotation, scaling, inversion, squeezing, shearing, projection and more and more.

As a convention, we'll be marking vectors with lowercase letters and matrices with uppercase letters.

 
#### Matrix Multiplication as a dot product
The easiest way to look at a matrix is to look at it as a bunch of vectors. Depending on what we care about, we can either look at the columns or rows as vectors. 

**//TODO: 2x2 example**

##### Identity
##### Scale
##### Rotation matrices

1. in 2D
1. in 3D
	* Example: Vibrating a brick-phone in 3D.
	
#### The inverse matrix
### "The Full Stack"
#### Homogenous coordinates: Hacking 3d in 4d
#### Translation matrices
#### SRT (Scale-Rotate-Translate) operations
* Example: a pack of sharks swimming

### Really using normals
#### The cross product
#### Normals for lighting
* Example: Dot product for lighting

#### Normals for directions
                
                
                
# Making your software generate (aka That other math chapter)

1. Probability
	1. How artists use probability
	1. Some interesting properties of probability
		1. Always sum to 1
		1. Expectation, Average
			* Example: Flocking, via finding the average of points.
		1. Probability as a density
			* Example: Cell colony in 2D
1. Randomness
	1. Different types of random functions: Uniform, Gaussian, etc
		* Example: Circle packing using `ofRandom`
	1. Markov Chains
		 * Example: ?
1. Noise
	1. From random numbers to streams:
		* Example: White noise
	1. Octaves: The construction of white noise
	1. Building things with `ofNoise`
		* Example: FDM
		* Example: Generative terrain
		* Example: Flickering lights


