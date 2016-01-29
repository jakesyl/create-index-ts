# create-index

[![NPM version](http://img.shields.io/npm/v/create-index.svg?style=flat-square)](https://www.npmjs.org/package/create-index)
[![Travis build status](http://img.shields.io/travis/gajus/create-index/master.svg?style=flat-square)](https://travis-ci.org/gajus/create-index)
[![js-canonical-style](https://img.shields.io/badge/code%20style-canonical-blue.svg?style=flat-square)](https://github.com/gajus/canonical)

`create-index` program creates (and maintains) ES6 `./index.js` file in target directories that imports and exports sibling files and directories.

## Usage

### Using CLI Program

```sh
npm install create-index

create-index --help

Options:
  --update-index  Recursively iterates target directories looking for "index.js"
                  files that start with "'create index';\n" (create-index index
                  file). Updates found index files. Does not create new index
                  files.                              [boolean] [default: false]

Examples:
  create-index ./src ./src/utilities        Creates or updates an existing
                                            create-index index file in the
                                            target (./src, ./src/utilities)
                                            directories.
  create-index --update-index ./src         Finds all create-index index files
  ./tests                                   in the target directories and
                                            descending directories. Updates
                                            found index files.
```

### Using `create-index` Programmatically

```js
import {
    writeIndex
} from 'create-index';

/**
 * @type {Function}
 * @param {Array<string>} directoryPaths
 * @throws {Error} Directory "..." does not exist.
 * @throws {Error} "..." is not a directory.
 * @throws {Error} "..." unsafe index.
 * @returns {boolean}
 */
writeIndex;
```

Note that the `writeIndex` function is synchronous.

```js
import {
    findIndexFiles
} from 'create-index';

/**
 * @type {Function}
 * @param {string} directoryPath
 * @returns {Array<string>} List of directory paths that have create-index index file.
 */
findIndexFiles;
```

### Gulp

Since [Gulp](http://gulpjs.com/) can ran arbitrary JavaScript code, there is no need for a separate plugin. See [Using `create-index` Programmatically](#using-create-index-programmatically).

```js
import {
    writeIndex
} from 'create-index';

gulp.task('create-index', () => {
    writeIndex(['./target_directory']);
});
```

Note that the `writeIndex` function is synchronous.

## Implementation

`create-index` program will look into the target directory.

If there is no `./index.js`, it will create a new file, e.g.

```js
'create index';
```

Created index file must start with `'create index';\n\n`. This is used to make sure that `create-index` does not accidentally overwrite your local files.

If there are sibling files, index file will `import` them and `export`, e.g.

```sh
children-directories-and-files git:(master) ✗ ls -lah
total 0
drwxr-xr-x   5 gajus  staff   170B  6 Jan 15:39 .
drwxr-xr-x  10 gajus  staff   340B  6 Jan 15:53 ..
drwxr-xr-x   2 gajus  staff    68B  6 Jan 15:29 bar
drwxr-xr-x   2 gajus  staff    68B  6 Jan 15:29 foo
-rw-r--r--   1 gajus  staff     0B  6 Jan 15:29 foo.js
```

Given the above directory contents, `./index.js` will be:

```js
'create index';

import bar from './bar';
import foo from './foo.js';

export {
    bar,
    foo
};
```

When file has the same name as a sibling directory, file `import` takes precedence.

Directories that do not have `./index.js` in themselves will be excluded.

When run again, `create-index` will update existing `./index.js` if it starts with `'create index';\n\n`.

If `create-index` is executed against a directory that contains `./index.js`, which does not start with `'create index';\n\n`, an error will be thrown.