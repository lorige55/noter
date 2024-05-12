# noter

Hi there! Thanks for checking out this repo! Noter is a Note-App specifically for Students. It's, as you can see, still in development. Therefor it has many bugs and lacks a lot of features. If (I don't know why) you'd want to try it out, just open https://noter.yeomid.com. However if you want host it for yourself or build your own features on top of my code, follow the following instructions. 

## Run local instance of Noter
### 1. Install required Software

To be able to run Noter and host it for yourself, you need to make sure that you've installed [Node.js](https://nodejs.org/en) and [Git](https://git-scm.com/downloads)

If you've installed them, just proceed to the next step.

### 2. Download the code

Since you've installed Git, you can easily download the code of Noter with this command:

```
git clone https://github.com/lorige55/noter
```

Great! Now make sure the new directory "noter" is at the place you want it to be. Because this is where it will stay.

### 3. Install NPM Packages

Now go ahead and change into the directory.

```
cd noter
```

And now run

```
npm install
```

### 4. Build and Host

Now just run the build command

```
npm run build:watch
```

To start a web server that hosts the website, run

```
npm run preview
```
### Update!

I constantly commit new code to this repo so I would recommend regularly running

```
git pull
```
