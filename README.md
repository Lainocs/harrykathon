# Harrykathon

## Requirements

- NodeJS (>= v18)
- Yarn

## Install project

- Go in each subproject and run `npm install`

### Alternatively

If you run a UNIX system, you can run `bash QuickInstall.sh` in the root folder, it will install all the projects.

## Start the project

The project has multiple subprojects, you must start them all.

- In `potion-landing` -> `npm run dev`
- In `potion-front` -> `npm run serve`
- In `potion-back` -> `npm run start`

It runs on `http://localhost:5173`.

#### Alternatively

You can run `npm run start` in the root folder, it will start all the projects.

## Stop the project

In root folder, run `bash Stop.sh`, it kills the processes on the ports 3000, 808* and 51*. (wildcards used in case you started the processes multiple time by spamming start command).

# Authors

- De Garrigues Nicolas
- Louveau Tom
- Quintin Florent
- Braunschweig Malo
- Cousin-Alliot Julien
