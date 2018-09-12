class: center, middle

# Raytracing in Haskell

![](/cover.png)

---

class: middle

# Our aim

.full-width[![](/images/light.png)]

---

### I heard we need light sources in a raytracer?

--

####  No

--

### What about complex shapes?

--

#### No

--

### Anything other than spheres and simple light rules?

--

#### No

--

### But I heard Haskell is really bad with, like, fast stuff?

--

#### No


---

# How will we get there?

* What is the raytracer doing?
* Massiv undertaking
* Avenues for optimization
* How to profile performance
* To the Core of it
* Wrap-up


---

# How our raytracer works

.full-width[![](/images/trace.svg)]

---

### Core of the raytracer

.third-width[![](/images/trace.svg)]

```haskell
color ray{Ray r g b direction} surface = 
  case scatter ray surface of
    Nothing -> skyColor
    Just ray2@(Ray r2 g2 b2 direction2) -> color (Ray r*r2 g*g2 b*b2 direction2)
```

--

```haskell
scatter ray{Ray r g b direction) (Metal albedo fuzz normal) = 
  if dot (reflect direction) normal > 0
    then Just $ Ray r g b (reflect direction + fuzz * randomVector)
    else Nothing
```

--

```haskell
colorAtPixel x y dx dy n = (foldl' colorAtSample (0, 0, 0) [1..n]) / n
  where colorAtSample sampleId = color $ getRayThrough (x + dx * n) (y + dy * n)
```

--

That's basically it

???

Raytracer is an interesting performance testing subject, because we don't know
beforehand where the ray will go. Also, we need A LOT of rays to get a useful image


---

# Why Massiv

- Really natural idioms
- Promises great performance
- I don't want to do scheduling, work sharing and nitty gritty details myself

--

```haskell
newtype OurArray a = OurArray (Array B Ix1 a)
newtype SomeInts = Array P Ix2 Int
newtype Delayed = Array D Ix1 Int

λ> let vec = makeVectorR D Seq 10 id
λ> vec
(Array D Seq (10)
  [ 0,1,2,3,4,5,6,7,8,9 ])

λ> let vec2 = fmap (^ (2::Int)) vec
λ> evaluateAt vec2 4
16

λ> let vec2U = computeAs U vec2
λ> vec2U
(Array U Seq (10)
  [ 0,1,4,9,16,25,36,49,64,81 ]) 
```

???

You should not make the mistake I made of having a delayed array all the way to the top.
If a delayed variable is shared, it will be computed twice!

---

class: center, middle

# Thank you

### @monad_cat
