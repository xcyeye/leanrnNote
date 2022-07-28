# commander

```js
const { Command } = require('commander');
const program = new Command();


program
    .option('-d, --debug', 'output extra debugging')
    .option('-s, --small', 'small pizza size')
    .option('-p, --pizza-type <type>', 'flavour of pizza');

program.parse(process.argv);

const options = program.opts();
```

`option('-p, --pizza-type <type>', 'flavour of pizza');`这种情况下，参数必须存在，否则会直接退出

`option('-d, --debug', 'output extra debugging')`这种，值为true或者undefined，如果输入`xxx -d`，那么`options.debug`值便为TRUE



```js
program
    .option('-c, --cheese <type>', 'add the specified type of cheese', 'blue');

program.parse();
```

> `option('-c, --cheese <type>', 'add the specified type of cheese', 'blue');`这种情况blue就是默认值，输入`xxx`，就可以打印出`blue`，但是输入`xxx -c`，还是会直接退出，缺少参数



