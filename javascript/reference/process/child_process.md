# Child Process

* https://www.youtube.com/watch?v=9o8B3L0-d9c

```js
// all node scripts only have and run in a single process

var arguments = process.argv

process.stdout.write('hello ' + arguments)

var count = 0
setInterval(function () {
  if (count++ > 10) process.exit()
}, 100)

var exec = require('child_process').exec

exec('cat index.js', funtion (err, stdout, stderr) {
  console.log('we got our catted file', stdout)
})

var spawn = require('child_process').spawn

// spawn('cat')

if (process.argv[2] === 'child') {
  console.log('Im inside the child')
}
else {
  var child = spawn(process.execPath, __filename, 'child')
  // child.stdout.on('data', function (data) {
  //   console.log('from child', data.toString())
  // })
  child.stdout.pipe(process.stdout)
  // above line is alternative to 4th argument to spawn()
  // { stdio: 'inherit' }
}
```

# example2

```js
var spawn = require('child_process').spawn

// each child has its own isolated variables (like modules)
var bears = 0
bears += 1

if (process.argv[2] === 'child') {
  var net = require('net')
  var pipe = new net.Socket({ fd: 3 }) // stdio[3]
  pipe.write("i'm done")
}
else {
  var child = spawn(process.execPath, __filename, 'child', {
    // stdio: 'inherit'
    /* otherwise:
    stdio: [null, null, null] // [stdin, stdout, stdterr]
    // null uses the defaults to create a pipe for each of them
    // custom channels are possible, thus:
    */
    stdio: [null, null, null, 'pipe']
  })
  child.stdio[3].on('data', function (data) {
    if (data.toString() === "i'm done") {
      console.log('RIP')
      child.kill()
    }
  })
  console.log('parent', bears) // prints 1
}
```
