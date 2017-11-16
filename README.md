# AlphaGOZero
This is a trial implementation of DeepMind's Oct19th publication: [Mastering the Game of Go without Human Knowledge](https://www.nature.com/articles/nature24270.epdf?author_access_token=VJXbVjaSHxFoctQQ4p2k4tRgN0jAjWel9jnR3ZoTv0PVW4gB86EEpGqTRDtpIz-2rmo8-KG06gqVobU5NSCFeHILHcVFUeMsbvwS-lxjqQGg98faovwjxeTUgZAUMnRQ).

---

## Useful links:

[All DeepMind’s AlphaGO games](http://www.alphago-games.com)

[GoGOD dataset, $15](https://gogodonline.co.uk)

[KGS >=4dan, FREE](https://www.u-go.net/gamerecords-4d/)

[Youtube: Learn to play GO](https://www.youtube.com/watch?v=xMshtO8h7RU)

[repo: MuGo](https://github.com/brilee/MuGo)

[repo: ROCAlphaGO](https://github.com/Rochester-NRT/RocAlphaGo)

[repo: miniAlphaGO](https://github.com/yotamish/mini-Alpha-Go)

[repo: resnet-tensorflow](https://github.com/ritchieng/resnet-tensorflow)

[repo: leela-zero (c++ popular 9dan Go A.I. owned by Mozilla)](https://github.com/gcp/leela-zero)

[repo: reversi-alpha-zero (if you like reversi(黑白棋))](https://github.com/mokemokechicken/reversi-alpha-zero)

[repo: Sabaki](https://github.com/yishn/Sabaki/releases)

---

# Set up

## Install requirement

python 3.6

```
pip install -r requirement.txt
```

## Download Dataset (kgs 4dan)

Under repo's root dir

```
cd data/download
chmod +x download.sh
./download.sh
```

## Preprocess Data

*It is only an example, feel free to assign your local dataset directory*

```
python preprocess.py preprocess ./data/SGFs/kgs-*
```

## Train A Model

```
python main.py --mode=train --force_save —-n_resid_units=20
```

## Play Against An A.I. (currently only random A.I. is available)

```
python main.py --mode=gtp —-policy=random

2017-11-16 02:19:45,274 [20046] DEBUG    Network: Building Model Complete...Total parameters: 1581959
2017-11-16 02:19:45,606 [20046] DEBUG    Network: Loading Model...
2017-11-16 02:19:45,615 [20046] DEBUG    Network: Loading Model Failed
2017-11-16 02:19:46,702 [20046] DEBUG    Network: Done initializing variables
GTP engine ready
clear_board
=


showboard
   A B C D E F G H J K L M N O P Q R S T
19 . . . . . . . . . . . . . . . . . . . 19
18 . . . . . . . . . . . . . . . . . . . 18
17 . . . . . . . . . . . . . . . . . . . 17
16 . . . . . . . . . . . . . . . . . . . 16
15 . . . . . . . . . . . . . . . . . . . 15
14 . . . . . . . . . . . . . . . . . . . 14
13 . . . . . . . . . . . . . . . . . . . 13
12 . . . . . . . . . . . . . . . . . . . 12
11 . . . . . . . . . . . . . . . . . . . 11
10 . . . . . . . . . . . . . . . . . . . 10
 9 . . . . . . . . . . . . . . . . . . .  9
 8 . . . . . . . . . . . . . . . . . . .  8
 7 . . . . . . . . . . . . . . . . . . .  7
 6 . . . . . . . . . . . . . . . . . . .  6
 5 . . . . . . . . . . . . . . . . . . .  5
 4 . . . . . . . . . . . . . . . . . . .  4
 3 . . . . . . . . . . . . . . . . . . .  3
 2 . . . . . . . . . . . . . . . . . . .  2
 1 . . . . . . . . . . . . . . . . . . .  1
   A B C D E F G H J K L M N O P Q R S T
Move: 0. Captures X: 0 O: 0

None
=

play Black B1
= (1, (2, 1))

```

## Play in Sabaki

The support for Sabaki is **available**. Go to [Sabaki-releases](https://github.com/yishn/Sabaki/releases) and grab the latest version for your device.

1. Open Sabaki, play around with **'View'** and checkout **'Engine'**. Go to 'Manage engine' and add ```/path/to/main.py``` to 'path' with argument ```--mode=gtp```.
2. Open ```/path/to/main.py``` with your IDE/CMD, etc. Change the bash bang command (i.e. ```#!/usr/.../python```) to the output of ```which python``` on your device.
3. Go back to Sabaki, checkout **'Engine'** and toggle **'Attach..'**. Select A.I. as either ```W``` or ```B```, or both.
4. Click Ok. All set!
5. By the same token, add GNU Go and Leela under ```engine``` folder to Sabaki as well!

![](/figure/Sabaki.png)

## Basic Self-play

Under repo’s root  dir

```
python main.py --mode=selfplay
```

---

## AlphaGo Zero Architecture:

* input 19 x 19 x 17: 7 previous states + current state player’s stone, 7 previous states + current state opponent’s stone, player’s colour
* 1. A convolution of 256 filters of kernel size 3 x 3 with stride 1
  2. Batch normalisation
  3. A rectifier non-linearity

**Residual Blocks**
* 1. A convolution of 256 filters of kernel size 3 x 3 with stride 1
  2. Batch normalisation
  3. A rectifier non-linearity
  4. A convolution of 256 filters of kernel size 3 x 3 with stride 1
  5. Batch normalisation
  6. A skip connection that adds the input to the block
  7. A rectifier non-linearity

**Policy Head**
* 1.A convolution of 2 filters of kernel size 1 x 1 with stride 1
  2. Batch normalisation
  3. A rectifier non-linearity
  4. A fully connected linear layer that outputs a vector of size 192^2 + 1 = 362 corresponding to logit probabilities for all intersections and the pass move

**Value Head**
* 1. A convolution of 1 filter of kernel size 1 x 1 with stride 1
  2. Batch normalisation
  3. A rectifier non-linearity
  4. A fully connected linear layer to a hidden layer of size 256
  5. A rectifier non-linearity
  6. A fully connected linear layer to a scalar
  7. A tanh non-linearity outputting a scalar in the range [ 1, 1]

---

# Credit:

*Brain Lee
*Ritchie Ng
