# simple-plotter

An attempt to make plotting as easy on a modern computer running clojure as it used to be on a ZX Spectrum. 

See my original blogpost about simple plotter:
http://www.learningclojure.com/2010/09/graphics-like-its-1979-how-to-get.html

Or the original rant:
http://johnlawrenceaspden.blogspot.co.uk/2009/11/behold-in-its-full-glory-program-that-i.html

## Usage

```clojure
;; The classic ZX Spectrum sine drawing program

(use 'simple-plotter)
(create-window "ZX Graphics" 256 176)

;; 10 FOR X = 1 TO 255
;; 20 PLOT X, 88+80*SIN(X/128*PI)
;; 30 NEXT X

(doseq [x (range 256)]
  (plot x (+ 88 (* 80 (Math/sin (* Math/PI (/ x 128)))))))

;; Hey, let's have some axes as well

;; 40 INK 6
;; 50 PLOT 0,88: DRAW 255,0
;; 60 PLOT 127,0: DRAW 0,168

(ink gray)
(plot 0 88) (draw 255 0)
(plot 127 0) (draw 0 175)
```

Plot the graph of sine zx-style, conceding only that there are more pixels on a modern screen

```clojure
(use 'simple-plotter.core)

(create-window "sine")

;; sine graph
(doseq [x (range 1024)]
  (plot x (+ 384 (* 376 (Math/sin (* Math/PI (/ x 512)))))))

;; axes
(ink yellow)
(plot 0 384) (draw 1024 0)
(line 512 0 512 1024)


```
Alternatively, get the plotter to do all the scaling for you:

```clojure

(use 'simple-plotter.core)

(create-window "sine in sane coords" 200 100 black white -7.0 7.0 -1.0 1.0)
(doseq [x (range -7 7 0.01)]
  (draw-to x (Math/sin (* Math/PI x))))

;; axes
(ink lightgray)
(axes)

```

;; Barnsley's Famous Fern
;; generated by an iterated function system.

```clojure
(use 'simple-plotter)

(defn transform [[xx xy yx yy dx dy]]
  (fn [[x y]] [(+ (* xx x) (* xy y) dx)
               (+ (* yx x) (* yy y) dy)]))

(def barnsleys-fern '((2  [  0    0     0     0.16 0 0    ])
                      (6  [  0.2 -0.26  0.23  0.22 0 0    ])
                      (7  [ -0.15 0.28  0.26  0.24 0 0.44 ])
                      (85 [  0.85 0.04 -0.004 0.85 0 1.6  ])))

(defn choose [lst] (let [n (count lst)] (nth lst (rand n))))

(defn iteration [transforms]
  (let [transform-list (mapcat (fn [[n t]] (repeat n (transform t))) transforms)]
    (fn [x] ((choose transform-list) x))))

(def barnsley-points (iterate (iteration barnsleys-fern) [0 1]))

(create-window "Barnsley's Fern" 350 450)

(ink green)
(scaled-scatter-plot (take 10000 barnsley-points) 50 300 50 400 100)

```

```clojure
(use 'simple-plotter)

(defn draw-grid [step]
  (let [h (get-original-height) w (get-original-width)]
    (doseq [i (range 0 w step)]
      (plot i 0) (draw 0 h))
    (doseq [i (range 0 h step)]
      (line 0 i w i))))

(defn draw-grids [n colors]
  (when (and (<= n (get-original-width)) (seq colors))
    (ink (first colors))
    (draw-grid n)
    (recur (* n 2) (rest colors))))

(def g1 (create-window "grid1" 300 400))
(def g2 (create-window "grid2" 300 400))

(window! g1)
(draw-grids 2 (list red yellow green))

(window! g2)
(draw-grids 2 (repeat white))

(window! g1)
(ink blue)
(draw-grid 16)
```

```clojure
(use 'simple-plotter)

(def tinyrange '(1
                 1/10000000000 
                 1/100000000000000000000000 
                 1/100000000000000000000000000000000000000000000000
                 0.1 
                 0.00000000000000000000000001
                 0.000000000000000000000000000000000000000000000001))



(for [tiny tinyrange]
  (let [xcoords (range (- tiny) tiny (/ tiny 100))
        ycoords (map #(* tiny (Math/sin (* Math/PI (/ % tiny)))) xcoords)
        pairs (partition 2 (interleave xcoords ycoords))]

    (create-window (str "minute scales:" tiny) 400 300 white black  (- tiny) tiny (- tiny) tiny)
    (axes)
    (doseq [[x y] pairs] (draw-to x y))))

```




## License

Copyright © 2013 John Lawrence Aspden

Distributed under the Eclipse Public License, the same as Clojure.
