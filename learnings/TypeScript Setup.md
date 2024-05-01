## Install Typescript using NPM

`npm install typescript --save-dev`

## install ts-node for running TS code
`npm install ts-node

## setup nodemon config in nodemon.json
```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/**/*.spec.ts"],
  "exec": "ts-node ./src/server.ts"
}
```

## create directory to store code
`mkdir src`

