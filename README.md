
## PowerSchool Plugin Boilerplate

The boilerplate is designed to enable PowerSchool Plugin development with modern Javascript standards with Babel transformation. This boilerplate uses Webpack and Gulp to compile the code into a distribution plugin.zip file. 

1. Clone the boilerplate into a Project directory.
2. Rename the ps-plugin-boilerplate directory to the desired name of the application. 
	- `mv ps-plugin-boilerplate {{app_name}}`
3. cd into the `{{app_name}}` directory and delete .git in the directory. 
	- `cd {{app_name}}`
	- `rm -rf .git`
4. Open package.json and change the following to fit your app.
	- `name:`
	- `version:`
	- `description:`
	- `author:`
5. All `{{tags}}` should be renamed to the respective `tag_name`
6. Install all dependency from package.json
	- `npm i --save-dev`
	- or
	- `yarn install --dev`
7. Initialize git and push. 
	- `git init`
	- `git remote add origin git@github.com:User/UserRepo.git`
	- `git add .`
	- `git commit -m "first commit"`
	- `git push origin HEAD`
8. If supported initialize eslint
	- `eslint -init`
9. When development is finished use `gulp4-ps-tasks` to package the application.  
	- Standalone plugin 
		- Creates a `plugin.zip` with all dist directory contents.
	- Plugin with Image Server deployment
		- Creates a `plugin.zip` with web_root hooks.
		- Calls `gulp.config.json` and pushes `.js` and `.css` to selected image server.
10. `gulp4-ps-tasks` has several tasks. The boilerplate is only concerned with the two Orchestrator tasks. Orchestrators will call the other tasks as needed.
	- `gulp createPkgNoImage`
	- `gulp createPkgWithImage`
11. To include the SCSS file it is required to import it into the index.js so webpack can complie it.
	- `import '../sass/index.scss'`
12. To package images add them to the Images/{{App_name}}/ directory. Then referance like so.
	- `../../../images/{{app_name}}/my_image_file`

### Development
- In one tab, start the `webpack-serve` dev server with this command: `webpack-serve --config webpack.dev.config.js --require @babel/register`. Note that by running this command, you are also generating a version of the page fragments in `./src` which will load them from `http://localhost:8081`.
- After `webpack-serve` has started and the build finishes (that's important to let it finish), open a new Terminal instance and build the plugin: `gulp buildPreprocess`. This will copy over everything in `./plugin`, run it through `gulp-preprocess`, and then create a zip file of the plugin. This plugin will load everything from `./src` from `http://localhost:8081`, which means you can quickly see the changes you make in dev. Install this zip file on the PS test server, not live. When it comes time to deploy, follow the instructions below.
- NOTE: By installing a version of the plugin that loads all of your css/js from `http://localhost:8081`, that means the plugin will only work from the host that is running `webpack-serve`, which is your development machine. 

### Standalone Plugin

- Run cmd: `gulp createPkgNoImage`

### Plugin with Image Server

- Setup `gulp.config.json`
- Rename `image_server_name` to desired name
- Define `default_deploy_target:`
- Create and Deploy
	- Default : `gulp createPkgWithImage`
	- With Env: `gulp createPkgWithImage --env image_server_name`

### Structure
- `src`: Anything that Webpack should include in its build process should be included in this folder. 
- `plugin`: Should contain the `plugin.xml` file, and any files that don't need to be included by Webpack.

### Automatic page fragment generation

Your plugin will need to import the `.js` and `.css` files generated by your plugin. 

The `.js` and `.css` files that the webpack build produces will contain a hash in the filename. For example, the `index` bundle defined in the `webpack.common.babel.js` file will produce a file with a filename like `index.45lkjsdf08234.js` The hash is included in the filename so that when your code changes, the new hash value is included in the new filename. This will force your browser to clear the cache upon code changes, but only when the code changes.

Instead of manually copy-and-pasting the hash into the `<script>` and `<link>` tags, let webpack generate those tags for you by following this process:

- Create the correct folder structure in `./src`. If your page fragment's location would normally be `plugin/web_root/admin/students/`, then create the following directory structure and place the page fragment file in `./src/admin/students`. If the location of the page fragment is `plugin/web_root/wildcards`, then create the following directory structure and place the page fragment in `./src/wildcards/`. An example page fragment was included in this boilerplate under `./src/{{ps_category}}/{{ps_location}}/{{page}}.{{app_name}}.content.footer.txt`.
- Set the `output.publicPath` webpack option to the correct PowerSchool hostname. It's located in `webpack.prod.babel.js`. For example, `publicPath: https://ps.example.com/`. `webpack.dev.babel.js` sets `publicPath` to `http://localhost:8081`, which is the location of the `webpack-serve` dev server and shouldn't need to be changed. 
- In `webpack.common.babel.js`, create a `new HtmlWebpackPlugin({})` plugin instance for each page fragment you want to auto-generate. An example was provided in this boilerplate. The `{}` passed to `new HtmlWebpackPlugin({})` should contain the following options:
	- template: The "input" file that webpack will use to insert the `<script>` and `<link>` elements.
	- PS_URL: URL to your PowerSchool instance. Usable as a "context variable" within the `template` file by using this syntax: `<%= htmlWebpackPlugin.options.PS_URL %>`
	- filename: The "output" file that webpack generates which includes `<script>` and `<link`> elements that import the `.js` and `.css` bundles, also created by webpack.
	- chunks: Defines which webpack "chunks" should be included in the "output" file (defined by the `filename` option above). For example, suppose your `webpack.common.babel.js` file has the option `entry.index` set to your `index.js` file. This instructs webpack to create a "chunk" labeled `index`. Note that the chunk name `index` is different from the actual name of the output file -- that is defined by the `output.filename` options found in the `dev` and `prod` webpack configs. Webpack also creates a `vendor` chunk, which includes code imported from the `node_modules` folder. So, to include your code form the `index` chunk, and the `vendor` chunk, set the `chunks` option to `["vendor", "index"]`. Make sure `vendor` comes before `index`, and always include the `vendor` chunk in this setting.
	- inject: Should webpack generate a complete HTML file structure? Leave this set to `false` most of the time unless you're creating your own page.
 See the [html-webpack-plugin options](https://github.com/jantimon/html-webpack-plugin#options) for more on what these options do.

 ### Troubleshooting
- If your JS is code loading (i.e., you can see it getting loaded in the DevTools:Network tab in Chrome), but it's not running, make sure the `chunks` option in your `new HtmlWebpackPlugin({})` config includes the `vendor` chunk.

