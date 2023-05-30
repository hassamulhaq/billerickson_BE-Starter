# billerickson_BE-Starter
A starter theme for Hybrid WordPress Themes. BE Starter is a copy of our Cultivate Starter theme, but without all the food-blogger-specific functionality.

##### Official repo: https://github.com/billerickson/BE-Starter

Article: `https://www.billerickson.net/hybrid-wordpress-theme-starter/`

A hybrid WordPress theme uses theme.json to define styles and customize the block editor while also using traditional PHP template files.

Hybrid themes leverage the block editor for content but not for building the theme itself. Block themes use the new Site Editor for building and customizing the theme directly in the block editor.

At CultivateWP we’ve built hybrid themes since before the block editor officially shipped in WordPress 5.0, and have built hundreds of hybrid themes for clients. We have an internal starter theme we use on all projects, which is continually updated with new features, improvements, and fixes with each major WP release.

BE Starter is a copy of our Cultivate Starter theme, but without all the food-blogger-specific functionality.
By the way, we’ll be growing our development team soon, so if you enjoy working on hybrid WordPress themes, let me know!

### Starter theme features

#### Define colors and styles with theme.json
We use a fully tokenized design process so every key in theme.json maps to a Figma design token.

The designers will customize the tokens to create a site’s style guide (ex: change all the brand colors, tweak the box shadows, adjust the font sizes). As they design the site, everything will be styled using these tokens.

When it’s time to develop the site, the developer customizes the theme.json file based on the settings in Figma Design Tokens. Then it’s simply a matter of styling elements with the appropriate tokens. For instance, h1 might use the “Colossal” font size while h2 uses “Huge”.

If custom font families are required, we’ll use a slug of “primary” and register them like this. We try to use primary/secondary/etc instead of overly-specific names for colors and font families so that when we redesign a site, we don’t end up with something like .has-blue-background-color { color: red; }

We use a contrast checker to determine which brand colors should use “foreground” (a dark color) or white as the text color. Make sure you edit _blocks-core.scss and include any colors here that should use white text.

Use the settings.custom.layout section of theme.json to set:
- content is the content area width used for Content and Content Sidebar layouts. It’s also used for the content width inside of a group block
- wide is the site’s max width used in the site header, site footer, the content area for the “Full Width Content” layout, and the “wide” alignment option
- sidebar is the sidebar width
- padding is the left/right padding on the content area
- block-gap is the space between blocks (ex: paragraphs and headings)
- block-gap-large is the top/bottom padding on full width groups with a background color, top/bottom margin on separator block, top/bottom padding on site-inner, and in a few other places that need a larger space than block-gap.

#### Build custom blocks using ACF and block.json
I have another article that goes into more detail about _building ACF blocks with block.json_.
`https://www.billerickson.net/building-acf-blocks-with-block-json/`

The starter theme will make it even easier for you to create these blocks:
- It will automatically register every block in the /blocks folder
- It will register the style block-{block name} if a style.css file exists in the block’s folder
- It will include the init.php file if one exists

Most of your blocks will only need a block.json file to register the block, a render.php file for the markup of the block, and a style.css file to style the block.

If you have additional code you need to load, you can create an init.php file which will be loaded on the init hook with a priority of 5. For instance, in the _social links block_ I’ve included in the starter theme, the init.php file has a function for retrieving all of the site’s social links from Yoast SEO.
`https://github.com/billerickson/BE-Starter/tree/master/blocks/social-links`

The blocks built in this method are totally self-contained. When it’s time to redesign a site, you can copy the entire /blocks directory to the new theme and everything will just work.

The blocks are styled with standard CSS (no SASS compiling) and CSS variables generated from the theme.json file. Ex:background: var(--wp--preset--color--backdrop);

The block’s CSS file does not pass through WordPress’ filter to auto-prepend them with .editor-styles-wrapper so you may need to add that class on your own for editor specific styling.

#### Block Areas for additional editable content
We use “block areas” to bring the block editor to global areas of the site (ex: sidebar, after post).

While you could use block-based widget areas in a similar way, I prefer a CPT because we add an admin body class of .block-area-{block area name} and can _adjust the content width_ to match how it will render on the frontend, so the “Sidebar” block area is 336px wide.
`https://github.com/billerickson/BE-Starter/blob/master/assets/scss/editor-layout.scss#L6-L19`
The theme includes a custom post type called “Block Areas” which appears under the Appearance section of the WP backend. It includes an ACF field group for linking a block area post to a theme-registered block area (sidebar, after-post, before-footer, 404).

To create the sidebar block area:
- Go to Appearance > Block Areas.
- Click “Add New” and create a block area with any name (ex: “Sidebar”). In the right column, under “Assigned To” select “Sidebar”.
- Publish the post and reload the page. You’ll see the content area now matches the width of the actual sidebar in the theme ( settings.custom.layout.sidebar in theme.json).

You can add/edit/remove the registered block areas in inc/block-areas.php. To display a block area in your theme, use Block_Areas\show( $block_area_name ).

#### SASS compiled using Node.js
Here’s a guide with more information on the _SASS Compile Setup_ `https://sprucecss.com/blog/the-simplest-sass-compile-setup/`.

Once you have the theme set up in your local environment, run npm install to install the SASS package. You can then run npm run sass-dev to automatically compile expanded CSS as changes are made to the SASS partials.

When you’re ready to commit your changes to a repo or push the theme live, you can run `npm run sass-prod` which compiles minified CSS.

#### Use be_icon() for loading inline SVGs
Include your SVG files in assets/icons/utility or create additional icon folders (ex: assets/icons/category). I recommend using _svgo_ or _SVGOMG_ to minify and clean them up.

Make sure the SVGs *do not* have a width and height attribute, but instead use the viewBox, and have the same width and height (ie: square artboard). Example:

`https://github.com/svg/svgo`
`https://jakearchibald.github.io/svgomg/`

```
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">

```

You can display these icons using the be_icon() function. For instance, if you want to display a 24×24 facebook icon, use be_icon( [ 'icon' => 'facebook', 'size' => 24 ] );

In addition to making it easy to load inline SVGs, this function includes smart caching to minimize the page load time.

Let’s say you have a heart icon that appears next to every post title on an archive page to save it. If you were to use an inline SVG, you’d load the full markup of that SVG every time that SVG appears.

Instead, when you call the “heart” icon with be_icon() it outputs `<svg><use href="#utility-heart"></use></svg>` and then loads the heart icon once in the site footer with that ID.

If you have a complicated SVG that is having issues displaying with this smart caching (usually because it has its own IDs inside it) you can include 'force' => true to skip this and load the full markup inline.

_See the be_icon()_ function for all the available settings.
`https://github.com/billerickson/BE-Starter/blob/master/inc/helper-functions.php#L97-L126`

#### Full Site Editing blocks have been removed
Inside the `editor.js` file we’re unregistering a lot of core WP blocks that are related to full site editing. We’re also removing blocks that do the same thing as our custom ACF blocks. For instance, the “Query Loop” block is replaced by our own custom _Post Listing block_, and “Media Text” is replaced by a “Content & Image” ACF block.
`https://github.com/billerickson/BE-Starter/blob/master/assets/js/editor.js`
`https://cultivatewp.com/resource-center/post-listing-block/`

#### Login Logo
Inside inc/login-logo.php you can customize the $logo_path, $logo_width, and $logo_height variables, and it will update the logo on the WP login page for you.

#### Summary
The hybrid theme approach has radically improved our design and development process. Our new Cultivate Pro service is only possible because of theme.json, Figma design tokens, and self-contained, reusable ACF blocks.

We’re able to build a fully custom WordPress theme in 8-12 hours rather than the 30-40hrs it used to take. Clients love how deeply integrated the block editor is in our themes. The color palettes use their brand colors, the font sizes and families use their style guide, and they’re able to edit all the content on the site – from simple posts, to the homepage, to the sidebar – using the block editor.

I’m excited to explore Full Site Editing and watch it mature over the next few years, but our team will be using hybrid WordPress themes for client work for the foreseeable future.

