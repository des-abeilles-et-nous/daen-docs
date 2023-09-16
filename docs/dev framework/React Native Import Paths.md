# React Native Import Paths

*Note: The following can also be achieved using babel and or ESLint configuration, but we provide here a simpler whilst robust method*

When your project directory structure becomes complex (and it may happen for [actually good reasons](React%20Native%20Project%20Structure.md), relative imports path can become a nightmare, because you have to carefully calculate them.

For instance let's say that:
- in app/features/onboarding/screen1/component1/myfile.js
- you need to use a shared component located in app/library/components/DaenComponents/DaenIcon/DaenIcon.js

`import DaenIcon from '../../../../library/components/DaenComponents/DaenIcon/DaenIcon';`

**The solution is to turn important folders into modules that Node can easily find.**

You achieve that by putting a minimal package.json file in directories you consider important enough, containing only
```
{
  "name": "the module name you want to set"
}
```

Depending on your project tree complexity, you will set this at a different level:

1.For instance you can decide to turn library/components into a 'components' module, and now you can `import DaenIcon from 'components/DaenComponents/DaenIcon/DaenIcon';`
2. Or you can decide to turn library/components/DaenComponents into a DaenComponents library and now you can `import DaenIcon from 'DaenComponents/DaenIcon/DaenIcon';`

**Obiouvsly, you must think ahead and not mixing both...**

But now, your imports are more readable, your project tree is opinionated as it marks clearly which groups are important, and you can move directories within the project tree with minimal impact.

Now you may wonder why there seems to be a duplicate in 'DaenComponents/DaenIcon/DaenIcon' ?

It's because this way, you can group in the same place both the JS code as well other files, tightly related, thus making the whole component self-contained and self supporting. Obviously, if the component consists in only one file, this is not needed.

**Ok, but now, my paths, even absolute, are quite ugly...**

Indeed. Let's say you decided to turn the library/components folder into a 'components' module (choice 1 above).
The `import DaenIcon from 'DaenComponents/DaenIcon/DaenIcon';` imports the 'DaenIcon' component from the 'DaenIcon.js' file staying in the 'DaenComponents/DaenIcon' sub folder of the 'library/components' folder (using the module 'components' shortcut... Phew....

You may want to simplify that by renaming your DaenIcon.js file into index.js. This way, you can use only the directory name (the same way index.html files are served by default by a web server.

This has 2 benefits
1. Now your import becomes `import DaenIcon from 'DaenComponents/DaenIcon';` 
2. You clearly communicate which file is the only 'public' file within de DaenIcon folder, and that the others are 'private'

 

