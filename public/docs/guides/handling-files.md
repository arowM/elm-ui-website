# Handling Files
Handling files are very common for web and mobile applications for a lot of tasks including:
- Selecting and uploading files to a server
- Selecting and showing images for the users local machine
- Downloading / saving files to the users local machine  

Elm-UI have the `Ui.Native.FileManager` module which exposes functions for selecting, downloading and reading files.

_Currently there are no official way for doing these things in Elm. The following patterns and solutions are Elm-UI only and uses native code._ 

## Selecting Files
Selecting files are implemented with an `input[type=file]` natively and exposed via the `openSingle` and `openMultiple` APIs.

An example of opening a file browser for selecting a single or multiple images:
```elm
-- Ui.FileManager.openSingle : String -> Task Never File
task = Ui.FileManager.openSingle "image/*" 

-- Ui.FileManager.openMultiple : String -> Task Never (List File)
task = Ui.FileManager.openMultiple "image/*"
```
If the user selects file(s) then the given `Msg` will be _called_ with the selected file(s), which has the following definition:
```elm
type alias File =
  { name : String
  , mimeType : String
  , size : Float
  , data : Data
  }
```

## Reading Files
After selecting a file you might want to read that as a string or as a [Data URI](https://en.wikipedia.org/wiki/Data_URI_scheme). There are conviniece functions for these: `readAsString` and `readAsDataURL` that will return a task.

Examples of reading a file:
```elm
-- Ui.FileManager.readAsDataURL : File -> Task Never String
task = Ui.FileManager.readAsDataURL file

-- Ui.FileManager.readAsString : File -> Task Never String
task = Ui.FileManager.readAsString file
```

## Uploading Files
Currently the **Http** package only allows sending **string or multipart data** and not binary or file data. 

The implementation however is using [FormData](https://developer.mozilla.org/en/docs/Web/API/FormData) which allows appending files. The trick is to create a `Http.StringData` that has a `File` object associated with it and passing that masqueraded as a string to the FormData object. By doing this we can send multipart forms with files.

To convert a `File` to FormData you can use the following code:
```elm
-- Ui.Native.FileManager.toFormData : String -> File -> Http.Data
data = Ui.Native.FileManager.toFormData "key" file
```

A full example of a file upload can be viewed [here](https://github.com/gdotdesign/elm-ui-guide/blob/master/examples/file-upload.elm).

## Downloading Files
Sometimes you have string data that you want to download to the users local machine for example a JSON or text file.

You can initiate a download with `download` function:
```elm
-- Ui.Native.FileManager.download : String -> String -> String -> Task Never String
task = Ui.Native.FileManager.download "my.json" "application/json" stringData
```
This functions uses the [chrome.fileSystem](https://developer.chrome.com/apps/fileSystem) to show the save as dialog for Chrome apps.