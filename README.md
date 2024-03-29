# SpotiPi
This program uses Spotify API and websockets to control Raspberry Pi and display song album art on LED strip lights.

## About SpotiPi
### Tech Stack
* Node
* Express.js
* PostgreSQL
* Liquid.js
* socket.io
* Spotify API
* Raspberry Pi


### K-Means Clustering Algorithm
A K-Means Clustering algorithm was implemented to determine the k dominant colors for a given image.  

Given a collection of points in D-dimensional space (music preferences, birthday, etc.), natural clusters form. Such is the case with dominant colors in an image. Clustering can be applied to a weighted graph G=(V,E) where each cluster has a distance from another cluster $d_{ij}$. To determine a clustering of points that maximizes the distance between clusters such that the distance among points in those clusters are minimized, the following algorithm is used:

1. Convert the image into a 2D array of [LAB](https://en.wikipedia.org/wiki/CIELAB_color_space) values.
2. At random, select k points from the 2D array which are the k centroids.
3. For each point p in the array:
   1. Determine the Euclidean distance from p to each centroid.
   2. Add the point p to the centroid cluster with the minimum Euclidean distance.
4. If more than 2 of the resulting clusters have length 0 (i.e. when the album art has very little color variation), restart kmeans to pick new random centroids. Else, replace clusters of length 0 with the cluster of max length to avoid recursing for a long period of time.
5. Determine the mean of each of the k clusters, these are the new centroids.
6. If the new centroids = old centroids, return the resulting cluster. Else, repeat from step 3.

## Setup
### Spotify API
1. Create an account and login to https://developer.spotify.com/
2. Create the SpotiPi app
3. Note the app Client ID and Client Secret
4. Set the redirect URI to `http://[RASPBERRY PI IP]:8888/`

### PostgreSQL Database
This database stores the results of the kmeans algorithm for a given Spotify albumID to avoid recalculations in the future.
1. Download PostgreSQL from https://www.postgresql.org/download/
2. Open Postgres. There should be 3 default databases, [root name], postgres, and template0. Double click `[root name]` or run `psql -p5432 "[root name]"` in the terminal.
3. Create the `spotipi` database.
  ```sql
  CREATE DATABASE spotipi;
  ```
4. Exit out of `[root name]` by running `\quit`. Open the newly create database by double clicking `spotipi` in Postgres or running  `psql -p5432 spotipi` in the terminal.
5. Create the table `colors`.
  ```sql
  CREATE TABLE colors (
     albumID VARCHAR(255) UNIQUE,
     colors VARCHAR(255)[]
   );
  ```
6. Exit using `\quit`

### Express Server on Raspberry Pi
1. `ssh [RASPBERRY PI IP]`
2. `git clone https://github.com/ajtadeo/Spotipi.git`
3. `cd Spotipi`
4. To use `node-canvas` and perform server-side image analysis, the following requirements must be met.
   * Node version 16.16.0. Check your node version with `node -v`.
   * Install the following packages:
     * MacOS:
       ```sh
       brew install pkg-config cairo pango libpng jpeg giflib librsvg 
       ```
     * Windows:
       ```sh
       sudo apt-get pkg-config cairo pango libpng jpeg giflib librsvg
       ```

   NOTE: If you get the following error `gyp: Call to 'node -e "require('nan')"' returned exit status 1 while in binding.gyp. while trying to load binding.gyp`, you must install the package nan via `npm i nan` before installing node-canvas.
5. `npm i`
6. `touch .env`
7. Edit `.env` to include the following variables:
```js
SPOTIFY_CLIENT_ID="s3cret" /* Generated from https://developer.spotify.com/ */
SPOTIFY_CLIENT_SECRET="s3cret" /* Generated from https://developer.spotify.com/ */
SPOTIFY_REDIRECT_URI="http://[HOSTNAME]:[PORT]/auth/callback/"
SESSION_SECRET="s3cret"
PGUSER="root" /* machine's currently logged in user */
PGPASSWORD="root password" /* machine's currently logged in user password */
PGDATABASE="spotipi"
PGHOST="localhost"
PGPORT="5432"
``` 
8. Open Postgres.
8. `pm2 start app.js`

### Using the App
1. Open `[RASPBERRY PI IP]:8888` in a web browser

## Resources
* https://dev.to/bogdaaamn/run-your-nodejs-application-on-a-headless-raspberry-pi-4jnn
* https://sonyarouje.com/2010/12/17/approach-to-count-dominant-colors-in-a-image/
* https://tatasz.github.io/dominant_colors/
* https://dordnung.de/raspberrypi-ledstrip/
* https://towardsdatascience.com/extracting-colours-from-an-image-using-k-means-clustering-9616348712be
