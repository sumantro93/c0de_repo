##Setup your server
(this would ideally be done with automated provisioning)
- add a deploy user with password-less ssh [see this gist](https://gist.github.com/learncodeacademy/3cdb928c9314f98404d0)
- install forever `npm install -g forever`

##Install flightplan
- `npm install -g flightplan`
- in your project folder `npm install flightplan --save-dev`
- create a flightplan.js file

```javascript
var plan = require('flightplan');

var appName = 'node-app';
var username = 'deploy';
var startFile = 'bin/www';

var tmpDir = appName+'-' + new Date().getTime();

// configuration
plan.target('staging', [
  {
    host: '104.131.93.214',
    username: username,
    agent: process.env.SSH_AUTH_SOCK
  }
]);

plan.target('production', [
  {
    host: '104.131.93.215',
    username: username,
    agent: process.env.SSH_AUTH_SOCK
  },
//add in another server if you have more than one
// {
//   host: '104.131.93.216',
//   username: username,
//   agent: process.env.SSH_AUTH_SOCK
// }
]);

// run commands on localhost
plan.local(function(local) {
  // uncomment these if you need to run a build on your machine first
  // local.log('Run build');
  // local.exec('gulp build');

  local.log('Copy files to remote hosts');
  var filesToCopy = local.exec('git ls-files', {silent: true});
  // rsync files to all the destination's hosts
  local.transfer(filesToCopy, '/tmp/' + tmpDir);
});

// run commands on remote hosts (destinations)
plan.remote(function(remote) {
  remote.log('Move folder to root');
  remote.sudo('cp -R /tmp/' + tmpDir + ' ~', {user: username});
  remote.rm('-rf /tmp/' + tmpDir);

  remote.log('Install dependencies');
  remote.sudo('npm --production --prefix ~/' + tmpDir + ' install ~/' + tmpDir, {user: username});

  remote.log('Reload application');
  remote.sudo('ln -snf ~/' + tmpDir + ' ~/'+appName, {user: username});
  remote.exec('forever stop ~/'+appName+'/'+startFile, {failsafe: true});
  remote.exec('forever start ~/'+appName+'/'+startFile);
});
```

##Deploy!
- `fly staging` or `fly production`

##Take it to the next level
[Run your node app as a system service so it runs after server reboots](https://gist.github.com/learncodeacademy/3a96aa1226c769adba39)
