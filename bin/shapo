#!/usr/bin/env node
var os = require("os");
var fs = require("fs");
var path = require("path");
var http = require("http");
var https = require("https");

var connect = require("connect");
var serveStatic = require("serve-static");
var serveIndex = require("serve-index");
var fallback = require("connect-history-api-fallback");
var minimist = require("minimist");
var debug = require("debug");
debug.enable("anywhere");

var exec = require("child_process").exec;
var spawn = require("child_process").spawn;

// see https://github.com/substack/minimist for more usage
// parse process.args
var argv = minimist(process.argv.slice(2), {
  alias: {
    silent: "s",
    port: "p",
    hostname: "h",
    dir: "d",
    log: "l",
    fallback: "f"
  },
  string: ["port", "hostname", "fallback"],
  boolean: ["silent", "log"],
  default: {
    port: 8000,
    dir: process.cwd()
  }
});

if (argv.help) {
  console.log("Usage:");
  console.log("  shapo --help // print help information");
  console.log("  shapo // 8000 as default port, current folder as root");
  console.log("  shapo 8888 // 8888 as port");
  console.log("  shapo -p 8989 // 8989 as port");
  console.log("  shapo -s // don't open browser");
  console.log("  shapo -h localhost // localhost as hostname");
  console.log("  shapo -d /home // /home as root");
  console.log("  shapo -l // print log");
  console.log("  shapo -f // Enable history fallback");
  process.exit(0);
}

// open URL in browser ,cross-platform
var openbrowser = url => {
  switch (process.platform) {
    case "win32": //Windows
      exec("start " + url);
      break;
    case "darwin": //macOS
      exec("open " + url);
      break;
    default:
      //Linux etc.
      spawn("xdg-open", [url]);
  }
};

// get IP address
var getIP = () => {
  var ifaces = os.networkInterfaces(); // like `$ ifconfig` in macOS
  var ip = "";
  for (var dev in ifaces) {
    ifaces[dev].forEach(function(details) {
      if (ip === "" && details.family === "IPv4" && !details.internal) {
        ip = details.address;
        return;
      }
    });
  }
  return ip || "localhost";
};

var log = debug("shapo");

var app = connect();
app.use(function(req, res, next) {
  res.setHeader("Access-Control-Allow-Origin", "*");
  if (argv.log) {
    log(req.method + " " + req.url);
  }
  next();
});
if (argv.fallback !== undefined) {
  console.log("Enable html5 history mode.");
  app.use(
    fallback({
      index: argv.fallback || "/index.html"
    })
  );
}
app.use(serveStatic(argv.dir, { index: ["index.html"] }));
app.use(serveIndex(argv.dir, { icons: true }));

var port = parseInt(argv._[0] || argv.port, 10);
var secure = port + 1;

var hostname = argv.hostname || getIP();

http.createServer(app).listen(port, function() {
  port = port != 80 ? ":" + port : "";
  var url = "http://" + hostname + port + "/";
  console.log("Running at " + url);
  if (!argv.silent) {
    openbrowser(url);
  }
});
