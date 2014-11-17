jquery-file-upload-gridfs-middleware
=============================

jquery-file-upload-gridfs-middleware. Based on https://github.com/aguidrevitch/jquery-file-upload-middleware
It uploads the uploads into a temporary dir ('uploadDir' in options) then stream into gridfs and remove the tempory file. 

* Add a new handler to store file into mongo gridfs

jQuery-File-Upload Express.js middleware. Based on the server code of [jQuery-File-Upload](https://github.com/blueimp/jQuery-File-Upload)

Installation:

```
    $ npm install jquery-file-upload-gridfs-middleware
```

Usage:

```javascript
    var express = require("express"),
        mongo = require('mongodb'),
        Grid = require('gridfs-stream'),
        upload = require('jquery-file-upload-gridfs-middleware');

    var app = express();

    var db = new mongo.Db('yourDatabaseName', new mongo.Server("127.0.0.1", 27017));
    var gfs = Grid(db, mongo);

    // configure upload middleware
    upload.configure({
        uploadDir: __dirname + '/public/uploads',
        mongoGfs: gfs,
        imageVersions: {
            thumbnail: {
                width: 80,
                height: 80
            }
        }
    });

    app.configure(function () {
        ...
        app.use('/upload', upload.gridFsHandler());
        app.use(express.bodyParser());
        ...
    });

```

On the frontend:

```html
   <input id="fileupload" type="file" name="files[]" data-url="/upload" multiple>
   <script>$('#fileupload').fileupload({ dataType: 'json' })</script>
```

To prevent access to /upload except for post (for security)
```javascript
upload.configure({
    uploadDir: __dirname + '/public/uploads/',
    uploadUrl: '/uploads'
});

/// Redirect all to home except post
app.get('/upload', function( req, res ){
	res.redirect('/');
});

app.put('/upload', function( req, res ){
	res.redirect('/');
});

app.delete('/upload', function( req, res ){
	res.redirect('/');
});

app.use('/upload', function(req, res, next){
    upload.gridFsHandler({
        uploadDir: __dirname + '/public/uploads',
        mongoGfs: gfs
    })(req, res, next);
});
```

Overriding global configuration

```javascript

    app.use('/upload2', upload.gridFsHandler({
        uploadDir: __dirname + '/public/uploads',
        mongoGfs: gfs
    }));

```

More sophisticated example - Events

```javascript
        app.use('/upload', upload.gridFsHandler());

        // events
        upload.on('begin', function (fileInfo, req, res) { 
            // fileInfo structure is the same as returned to browser
            // { 
            //     name: '3 (3).jpg',
            //     originalName: '3.jpg',
            //     size: 79262,
            //     type: 'image/jpeg',
            //     delete_type: 'DELETE',
            //     delete_url: 'http://yourhost/upload/3%20(3).jpg',
            //     url: 'http://yourhost/uploads/3%20(3).jpg',
            //     thumbnail_url: 'http://youhost/uploads/thumbnail/3%20(3).jpg' 
            // }
        });
        upload.on('abort', function (fileInfo, req, res) { ... });
        upload.on('end', function (fileInfo, req, res) { ... });
        upload.on('delete', function (fileInfo, req, res) { ... });
        upload.on('error', function (e, req, res) {
            console.log(e.message);
        });
```

Dynamic upload directory and url, isolating user files:

```javascript
        upload.configure({
            imageVersions: {
                thumbnail: {
                    width: 80,
                    height: 80
                }
            }
        });

        app.use('/upload', function (req, res, next) {
            // imageVersions are taken from upload.configure()
           upload.gridFsHandlerr({
                uploadDir: function () {
                    return __dirname + '/public/uploads/' + req.sessionID
                },
                uploadUrl: function () {
                    return '/uploads/' + req.sessionID
                },
                mongoGfs: gfs
            })(req, res, next);
        });
```


Other options and their default values:

```javascript
{
    tmpDir: '/tmp',
    uploadDir: __dirname + '/public/uploads',
    uploadUrl: '/uploads',
    targetDir: uploadDir,
    targetUrl: uploadUrl,
    ssl: false,
    hostname: null, // in case your reverse proxy doesn't set Host header
                    // eg 'google.com'
    maxPostSize: 11000000000, // 11 GB
    minFileSize: 1,
    maxFileSize: 10000000000, // 10 GB
    acceptFileTypes: /.+/i,
    imageTypes: /\.(gif|jpe?g|png)$/i,
    imageVersions: {
        thumbnail: {
            width: 80,
            height: 80
        }
    },
    imageArgs: ['-auto-orient'],
    accessControl: {
        allowOrigin: '*',
        allowMethods: 'OPTIONS, HEAD, GET, POST, PUT, DELETE'
    }
}
```

## Contributors

   * [@homerquan](http://github.com/homerquan)
   * [@soomtong](http://github.com/soomtong)
   * [@gsarwohadi](https://github.com/gsarwohadi)
   * [@peecky](https://github.com/peecky)
   * [@tonyspiro](https://github.com/tonyspiro)
   * [@derjust](https://github.com/derjust)

## License
Copyright (c) 2014 [Homer Quan](http://www.homerquan.com)
Released under the [MIT license](http://www.opensource.org/licenses/MIT).
