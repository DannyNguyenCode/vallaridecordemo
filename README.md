# Vallari Decor — Custom Shopify Theme

A custom Shopify Online Store 2.0 theme built for **Vallari Decor**, an e-commerce business specializing in décor for weddings, events, homes, and commercial spaces.

This project was built as a custom storefront rather than modifying an existing Shopify theme. The goal was to take a visual design from concept to a functioning Shopify store while learning and implementing Shopify's Liquid templating system, theme architecture, product data model, variants, collections, Theme Editor, and deployment workflow.

The finished theme includes a responsive storefront, Shopify-powered product and collection data, product variants, cart functionality, custom informational pages, and merchant-editable content through the Shopify Theme Editor.

---

## Project Overview

The project came together in several stages:

1. Shopify development environment setup
2. Theme skeleton and scaffold
3. Converting static designs into Liquid
4. Connecting the theme to Shopify Admin
5. Making content editable through the Theme Editor
6. Setting up products, collections, and variants
7. Rendering Shopify data dynamically
8. Testing locally
9. Uploading the theme to Shopify
10. Publishing the completed storefront

Rather than treating Shopify as a traditional React or static website, this project was designed around Shopify's own architecture.

The storefront UI lives in the theme, while Shopify remains responsible for commerce data such as:

- Products
- Variants
- Prices
- Inventory
- Collections
- Cart data
- Customer accounts
- Pages
- Store settings

Liquid connects those two layers together.

---

# 1. Development Setup

Development began by creating a Shopify development store and setting up the local Shopify theme development environment.

The main tools used were:

- Shopify
- Shopify CLI
- Liquid
- HTML
- CSS
- JavaScript
- JSON templates
- Git / GitHub
- Cursor / VS Code

Shopify CLI provides the connection between the local theme files and the Shopify store.

During development, the theme can be run locally with:

```bash
shopify theme dev
```

This starts Shopify's local theme development server.

A local storefront can then be accessed through an address similar to:

```text
http://127.0.0.1:9292
```

The important distinction is that the local development server still communicates with the Shopify development store.

The theme is local, but product data, collections, variants, pages, and other commerce information come from Shopify.

---

# 2. Building the Theme Skeleton

The next step was creating the basic Shopify theme structure.

The project follows Shopify Online Store 2.0 conventions and is organized around directories such as:

```text
assets/
config/
layout/
locales/
sections/
snippets/
templates/
```

Each directory has a different responsibility.

## Layout

`layout/theme.liquid` provides the main HTML document surrounding the storefront.

It is responsible for the global page structure and loading shared theme resources.

Conceptually:

```liquid
<!doctype html>
<html>
  <head>
    {{ content_for_header }}
  </head>

  <body>
    {% sections 'header-group' %}

    <main>
      {{ content_for_layout }}
    </main>

    {% sections 'footer-group' %}
  </body>
</html>
```

The individual Shopify templates are rendered through:

```liquid
{{ content_for_layout }}
```

This allows the header and footer to remain global while Shopify dynamically renders the correct page template.

---

# 3. Sections, Snippets, and Templates

One of the most important parts of the project was understanding the relationship between Shopify's templates, sections, and snippets.

## Templates

Templates determine which sections Shopify should render for a particular type of page.

For example:

```text
templates/
├── index.json
├── product.json
├── collection.json
├── page.about.json
├── page.contact.json
└── page.faq.json
```

The About Us page uses:

```text
page.about.json
```

A template can reference a custom section:

```json
{
  "sections": {
    "main": {
      "type": "vallari-about-page",
      "settings": {}
    }
  },
  "order": [
    "main"
  ]
}
```

Shopify then looks for:

```text
sections/vallari-about-page.liquid
```

This separation made it possible to build custom layouts for different Shopify resources while still allowing Shopify to control routing and content.

## Sections

Sections contain the major pieces of the storefront.

Examples include:

```text
vallari-hero.liquid
vallari-categories.liquid
vallari-featured-products.liquid
vallari-main-product.liquid
vallari-about-page.liquid
vallari-contact.liquid
vallari-faq.liquid
```

Sections can contain:

- Liquid
- HTML
- CSS
- JavaScript
- Shopify schema

## Snippets

Reusable UI was moved into snippets where appropriate.

For example, a reusable product card can be rendered with:

```liquid
{% render 'vallari-product-card', product: product %}
```

The section controls the collection or product loop, while the snippet controls how each individual product card is displayed.

This avoids duplicating product-card markup across multiple sections.

---

# 4. Converting the Design to Shopify Liquid

The Vallari Decor storefront was first structured as a static front-end design before being converted into a dynamic Shopify theme.

Those designs contained normal HTML and styling but were not connected to Shopify.

The next stage was translating those static layouts into Shopify Liquid.

A static product title such as:

```html
<h1>Classic Bridal Acrylic Crystal Vase Filler Set</h1>
```

becomes:

```liquid
<h1>{{ product.title | escape }}</h1>
```

A hard-coded price becomes Shopify data:

```liquid
{{ product.price | money }}
```

A hard-coded image becomes:

```liquid
{{
  product.featured_image
  | image_url: width: 1200
  | image_tag
}}
```

A hard-coded product URL becomes:

```liquid
<a href="{{ product.url }}">
```

The same process was used throughout the storefront.

The goal was to preserve the visual design while replacing mock content with Shopify objects.

---

# 5. Working With Liquid

Liquid acts as the bridge between the theme and Shopify.

It allows the storefront to access Shopify objects such as:

```liquid
product
collection
cart
customer
shop
section
routes
```

Liquid was used for conditional rendering:

```liquid
{% if product.featured_image != blank %}
  ...
{% endif %}
```

Loops:

```liquid
{% for product in collection.products %}
  {% render 'vallari-product-card', product: product %}
{% endfor %}
```

Section settings:

```liquid
{{ section.settings.heading }}
```

Shopify routes:

```liquid
{{ routes.all_products_collection_url }}
```

And image transformations:

```liquid
{{
  section.settings.hero_image
  | image_url: width: 1800
  | image_tag:
    loading: 'lazy'
}}
```

This allowed the theme to remain dynamic rather than tying the storefront to hard-coded content.

---

# 6. Shopify Section Schema

A major part of making the theme usable by a store owner was adding Shopify schema to custom sections.

Instead of requiring a developer to change headings, images, links, or descriptions directly in Liquid, those values can be exposed to Shopify's Theme Editor.

For example:

```liquid
<h1>
  {{ section.settings.hero_heading | escape }}
</h1>
```

The corresponding schema might contain:

```json
{
  "type": "text",
  "id": "hero_heading",
  "label": "Heading",
  "default": "Details that turn spaces into moments."
}
```

Images can be configured with:

```json
{
  "type": "image_picker",
  "id": "hero_image",
  "label": "Hero image"
}
```

And links with:

```json
{
  "type": "url",
  "id": "hero_link",
  "label": "Link"
}
```

This became especially important for sections such as the About Us page, where multiple editorial images, headings, descriptions, and links can now be changed without editing source code.

---

# 7. Shopify Admin

The Shopify Admin acts as the content and commerce management layer for the theme.

Instead of storing product information directly inside the codebase, products are created and maintained through Shopify.

The Admin was used to configure:

- Products
- Product images
- Prices
- Variants
- Collections
- Pages
- Navigation
- Store settings

For informational pages such as About Us, Contact Us, and FAQ, Shopify pages were created first.

Custom JSON templates were then created in the theme:

```text
page.about.json
page.contact.json
page.faq.json
```

Once those templates are available to the active theme, the Shopify page can be assigned to the corresponding template.

This keeps the page itself inside Shopify while allowing the theme to determine its presentation.

---

# 8. Shopify Theme Editor

The Theme Editor provides the visual configuration layer for the custom theme.

Section schema makes it possible for the merchant to change storefront content without modifying Liquid.

For example, the Vallari About page allows the merchant to configure:

- Hero image
- Hero heading
- Introduction
- Brand story
- Editorial image
- Brand principles
- Home décor image and link
- Event décor image and link
- Closing call-to-action

The same pattern was used throughout the storefront.

This created an important separation:

```text
Theme code
    ↓
Defines what can be configured

Theme Editor
    ↓
Controls presentation/content settings

Shopify Admin
    ↓
Controls products, variants, collections and commerce data
```

The final theme therefore remains customizable without requiring future code changes for routine content updates.

---

# 9. Setting Up Shopify Product Data

Once the storefront structure was working, the next step was replacing sample content with real Shopify product data.

Products were created through Shopify Admin with information such as:

```text
Title
Description
Price
Images
Product category
Collections
Variants
Inventory
```

The theme does not need to know the name of every individual product.

Instead, Liquid asks Shopify for the appropriate data.

For example:

```liquid
{% for product in featured_collection.products %}
  {% render 'vallari-product-card', product: product %}
{% endfor %}
```

Adding another product to the collection therefore allows it to appear in the storefront without creating another hard-coded product card.

---

# 10. Collections

Collections were used to organize products into meaningful groups.

Instead of manually creating category cards with static images and text, the homepage can work directly with Shopify collections.

For example:

```liquid
{% for category in section.settings.collections %}
  <a href="{{ category.url }}">
    {{ category.title }}
  </a>
{% endfor %}
```

The merchant can select which collections should appear through the Theme Editor using a setting such as:

```json
{
  "type": "collection_list",
  "id": "collections",
  "label": "Featured collections",
  "limit": 8
}
```

This allows Shopify to remain the source of truth for the store catalogue.

---

# 11. Product Variants

Products can have multiple purchasing options.

For Vallari Decor, this can include options such as:

```text
Color
Size
Style
```

Shopify represents combinations of those options as **variants**.

For example:

```text
Clear / Small
Clear / Large
Gold / Small
Gold / Large
Silver / Small
Silver / Large
```

Each variant can have its own:

- Variant ID
- Price
- Availability
- Inventory
- SKU
- Selected options

The product page therefore cannot treat the product as a single static item.

The selected variant needs to determine what gets submitted to the cart.

---

# 12. Rendering Variant Data

Liquid provides the initial product and variant data when the page is rendered.

For example:

```liquid
{% for option in product.options_with_values %}
  ...
{% endfor %}
```

Variant selectors are generated based on the actual options configured in Shopify.

The selected variant ID is then submitted through the Shopify product form.

Conceptually:

```liquid
{% form 'product', product %}
  <input
    type="hidden"
    name="id"
    value="{{ product.selected_or_first_available_variant.id }}"
  >

  <input
    type="number"
    name="quantity"
    value="1"
  >

  <button type="submit">
    Add to cart
  </button>
{% endform %}
```

JavaScript is used on the client side to respond when customers change product options.

The UI can then update information such as:

- Selected variant
- Price
- Availability
- Add-to-cart state

The important architectural distinction is that Shopify owns the actual variant data.

The theme only renders and interacts with it.

---

# 13. Rendering Dynamic Storefront Data

By this stage, the storefront had moved away from sample data.

Homepage sections could render Shopify collections and products, while product pages rendered whichever Shopify product the customer navigated to.

For example:

```liquid
{{ product.title }}
{{ product.description }}
{{ product.price | money }}
```

and:

```liquid
{% for media in product.media %}
  ...
{% endfor %}
```

This means one product template can support the entire catalogue.

There is no need to create:

```text
product-vase.liquid
product-candleholder.liquid
product-wedding-decor.liquid
```

for every individual product.

Instead:

```text
Shopify product data
        ↓
product.json
        ↓
vallari-main-product.liquid
        ↓
Dynamic product page
```

The same principle applies to collections.

This was one of the biggest transitions in the project: moving from designing individual screens to designing reusable storefront systems.

---

# 14. Local Testing

During development, Shopify CLI was used to preview changes locally.

```bash
shopify theme dev
```

The local environment was used to test:

- Responsive layouts
- Navigation
- Product rendering
- Product variants
- Product forms
- Cart behavior
- Collections
- About Us
- FAQ
- Contact Us
- Theme settings
- Mobile layouts

Alternate page templates could also be tested locally while development was still in progress.

For example:

```text
/pages/about-us?view=about
```

This was useful before the custom template was assigned to the Shopify page.

---

# 15. Pushing the Theme to Shopify

Once local development was complete, the theme was uploaded to Shopify as an unpublished theme.

```bash
shopify theme push --unpublished
```

This creates a draft theme inside:

```text
Shopify Admin
→ Online Store
→ Themes
→ Draft themes
```

For this project, the uploaded development theme was named:

```text
Vallari Decor - Development
```

Using an unpublished theme provided an additional staging step between local development and production.

The workflow became:

```text
Local source code
       ↓
Shopify CLI
       ↓
Unpublished Shopify theme
       ↓
Theme Editor
       ↓
Final review
       ↓
Publish
```

This prevents unfinished development work from immediately affecting the live storefront.

---

# 16. Publishing the Storefront

Once the uploaded theme was reviewed and ready, it could be published through Shopify Admin.

Publishing makes the custom Vallari theme the store's active theme.

After publishing, the custom page templates can be assigned to their corresponding Shopify pages:

```text
About Us    → about
FAQ         → faq
Contact Us  → contact
```

The final storefront should then be tested again in production, including:

- Homepage
- Products page
- Individual product pages
- Product variants
- Add to cart
- Cart
- Search
- Customer account navigation
- About Us
- FAQ
- Contact Us
- Desktop navigation
- Mobile navigation
- Responsive layouts

The previous Shopify theme remains available in the theme library and can be used as a rollback option if necessary.

---

# Development Workflow

The complete workflow used for this project can be summarized as:

```text
Design
  ↓
Create Shopify theme scaffold
  ↓
Build static section structure
  ↓
Convert markup to Liquid
  ↓
Add section schema
  ↓
Create JSON templates
  ↓
Configure Shopify Admin
  ↓
Create products and collections
  ↓
Configure product variants
  ↓
Render Shopify data through Liquid
  ↓
Add client-side interactions
  ↓
Test using Shopify CLI
  ↓
Push as unpublished theme
  ↓
Configure and review in Theme Editor
  ↓
Publish
  ↓
Assign page templates
  ↓
Production testing
```

---

# What I Learned

This project provided hands-on experience with the complete Shopify theme development lifecycle.

The most important lesson was understanding that a Shopify storefront is not simply a static website with products added afterward.

The theme and Shopify platform have separate responsibilities.

**Shopify manages commerce data:**

```text
Products
Variants
Collections
Inventory
Customers
Cart
Pages
Store configuration
```

**The custom theme manages presentation:**

```text
Layout
Responsive design
Product presentation
Collection presentation
Navigation UI
Interactive behavior
Merchant-editable sections
Brand styling
```

**Liquid connects the two.**

Understanding this architecture made it possible to transform the original static Vallari Decor design into a reusable, data-driven Shopify storefront.

---

# Tech Stack

- Shopify Online Store 2.0
- Shopify CLI
- Liquid
- HTML5
- CSS3
- JavaScript
- JSON templates
- Shopify Theme Editor
- Shopify Admin
- Git / GitHub
- Cursor / VS Code

---

# Project Status

**Completed**

The custom Vallari Decor Shopify storefront has been built with dynamic Shopify data, custom page templates, product variant support, merchant-editable theme settings, responsive layouts, and a production deployment workflow.

---

## Author

Custom Shopify theme developed as an end-to-end Shopify theme development project for Vallari Decor.

---

# How I Set Up This Project

This project started as a Shopify development demo for Vallari Decor. Rather than modifying a completed Shopify theme, I set up a native Shopify Online Store 2.0 development environment and built the storefront on top of Shopify's Skeleton theme.

The theme uses Liquid and HTML for rendering, CSS for styling, JavaScript for storefront interactions, and JSON for Shopify templates and configuration. Shopify itself provides the commerce layer, including products, collections, variants, cart functionality, customers, pages, and Theme Editor configuration.

## 1. Development Store

I first created the Shopify store that would act as the development environment for the project.

The development store allowed me to work with real Shopify resources while developing the theme locally:

```text
Shopify Development Store
├── Products
├── Collections
├── Variants
├── Pages
├── Navigation
├── Cart
└── Theme Editor
```

This meant the theme could be developed locally without hard-coding Shopify commerce data into the project.

---

## 2. Local Repository

I used an existing GitHub repository as the working project and opened it in Cursor.

The local project was kept as the source-controlled version of the theme.

I preserved the existing `.git` directory rather than replacing it when bringing Shopify's theme scaffold into the repository.

Git and Shopify have separate responsibilities:

```text
Git / GitHub
    ↓
Versions the source code

Shopify
    ↓
Runs the store and stores commerce data
```

A Shopify theme sync is therefore not a replacement for committing the source code to Git.

---

## 3. Shopify CLI

I installed Shopify CLI globally:

```bash
npm install -g @shopify/cli@latest
```

I verified the development tools with:

```bash
node --version
npm --version
git --version
shopify version
```

Installing Shopify CLI globally made the `shopify` commands available from different projects while the actual theme files remained inside the Vallari Decor repository.

---

## 4. Shopify Skeleton Theme

I used Shopify's Skeleton theme as the starting scaffold.

For a new Shopify theme, the scaffold can be created with:

```bash
shopify theme init
```

Because I already had an existing Git repository, I did not want the scaffold to replace the repository's Git metadata.

The Skeleton theme files were therefore brought into the existing project while preserving the original `.git` directory.

This provided the initial Shopify theme structure:

```text
assets/
blocks/
config/
layout/
locales/
sections/
snippets/
templates/
```

From this point forward, the storefront was developed as a native Shopify theme rather than as a React or Next.js application.

---

## 5. Initial Verification

Before building the Vallari storefront, I verified that the Shopify Skeleton theme worked correctly.

I started the development environment and confirmed:

- Shopify CLI connected to the store
- The starter theme rendered
- Liquid changes appeared locally
- CSS changes reloaded
- Theme Check could validate the project

This gave me a working baseline before replacing the Skeleton presentation with the Vallari design.

---

## 6. Local Development and Theme Editor Sync

During development, I connected the local project to the Shopify store using:

```powershell
$vallariStore = "niceguywebdesign-vallari-decor-demo.myshopify.com"

shopify theme dev --store $vallariStore --theme-editor-sync
```

This started the local development server while also allowing Theme Editor configuration changes to synchronize with the local project.

The local storefront was available at an address similar to:

```text
http://127.0.0.1:9292/
```

I kept the Shopify CLI process running while developing.

My development workflow was:

```text
Cursor
   │
   │ Liquid / CSS / JavaScript
   ▼
Local Shopify Theme
   │
   │ Shopify CLI
   ▼
Development Theme
   │
   ├── Shopify product data
   ├── Collections
   ├── Variants
   └── Theme Editor settings
```

This allowed me to write code locally while still working against actual Shopify data.

---

## 7. Dividing Work Between Cursor, Theme Editor, and Shopify Admin

One of the important parts of the setup was understanding where different changes should be made.

### Cursor

I used Cursor for changes to the actual theme source:

```text
Liquid
HTML
CSS
JavaScript
JSON templates
Section schema
Snippets
```

### Shopify Theme Editor

I used the Theme Editor for merchant-editable presentation settings such as:

```text
Hero content
Images
Featured collections
Featured products
Section settings
Section configuration
```

### Shopify Admin

I used Shopify Admin for actual store resources:

```text
Products
Variants
Collections
Pages
Menus
Product media
```

The resulting separation was:

```text
CURSOR
Theme implementation
      │
      ▼
THEME EDITOR
Presentation configuration
      │
      ▼
SHOPIFY ADMIN
Commerce and store data
```

This became the foundation for the rest of the project.

---

## 8. Saved Theme Configuration

I also learned that Shopify section code and Shopify's saved section configuration are separate.

For example:

```text
sections/vallari-hero.liquid
```

defines how the hero works and what settings are available.

But:

```text
templates/index.json
```

records the homepage section instances, their order, and their saved template settings.

Similarly:

```text
sections/header.liquid
```

contains the header implementation, while:

```text
sections/header-group.json
```

contains the configured header section instance and settings.

This distinction became important when synchronizing the local theme with Shopify's Theme Editor.

Changing or restoring a `.liquid` section does not necessarily restore its saved placement or configuration.

---

## 9. Development Sync Workflow

During a normal development session I used the following workflow:

```text
1. Start Shopify CLI
        ↓
2. Open local preview
        ↓
3. Open the matching Theme Editor
        ↓
4. Edit Liquid/CSS/JS in Cursor
        ↓
5. Save and verify the local storefront
        ↓
6. Configure content in Theme Editor
        ↓
7. Save Theme Editor changes
        ↓
8. Allow editor sync to update local configuration
        ↓
9. Run Theme Check
        ↓
10. Review Git changes
```

Before committing changes I could run:

```bash
shopify theme check
git status
git diff
```

This helped separate actual source-code changes from Shopify configuration changes.

---

## 10. Handling Local and Shopify Theme Conflicts

During development, I encountered situations where the local JSON configuration and Shopify's remote theme configuration differed.

This was especially important for files such as:

```text
templates/index.json
sections/header-group.json
sections/footer-group.json
config/settings_data.json
```

Rather than assuming the local or remote version was always correct, I compared the files before deciding which version to keep.

When a specific remote configuration needed to be recovered, Shopify CLI could pull only that file.

For example:

```powershell
shopify theme list --store $vallariStore

shopify theme pull `
  --store $vallariStore `
  --theme <theme-id> `
  --only "templates/index.json" `
  --nodelete
```

I treated development theme IDs as temporary identifiers and verified the correct theme before performing targeted pushes or pulls.

---

## 11. Moving From Scaffold to Custom Theme

Once the development environment and synchronization workflow were stable, I began replacing the Skeleton presentation with the custom Vallari storefront.

The project then progressed through:

```text
Shopify Skeleton
      ↓
Global layout
      ↓
Header / navigation
      ↓
Homepage sections
      ↓
Liquid conversion
      ↓
Shopify data
      ↓
Products and collections
      ↓
Product variants
      ↓
Custom page templates
      ↓
Theme Editor configuration
      ↓
Complete Vallari storefront
```

The Skeleton theme provided the Shopify-compatible foundation, but the storefront presentation and custom sections were built specifically for Vallari Decor.

---

## 12. Moving the Completed Theme to Shopify

After completing and testing the storefront locally, I uploaded the custom theme to Shopify as an unpublished theme:

```bash
shopify theme push --unpublished
```

The uploaded theme was named:

```text
Vallari Decor - Development
```

This created a draft version inside Shopify without immediately replacing the existing live theme.

The deployment path was:

```text
Local Vallari theme
        ↓
shopify theme push --unpublished
        ↓
Vallari Decor - Development
        ↓
Shopify Theme Editor
        ↓
Final review
        ↓
Publish
```

This provided a staging step between local development and the customer-facing storefront.

---

## 13. Publishing

Once the custom theme was complete, I published **Vallari Decor - Development** through:

```text
Shopify Admin
→ Online Store
→ Themes
→ Publish
```

Publishing made the custom Vallari theme the active Shopify storefront.

After the theme became active, the custom page templates could be assigned to the corresponding Shopify pages:

```text
About Us   → about
FAQ        → faq
Contact Us → contact
```

The previous Shopify theme remained in the theme library as a rollback option.

---

## Final Setup Architecture

The completed development environment can be summarized as:

```text
                    GitHub
                       ▲
                       │
                    Cursor
                       │
             Liquid / CSS / JS / JSON
                       │
                       ▼
              Shopify Theme Files
                       │
                  Shopify CLI
                       │
           ┌───────────┴───────────┐
           ▼                       ▼
     Local Preview           Shopify Theme
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
               Theme Editor    Shopify Admin    Store Data
                    │              │              │
                    └──────────────┼──────────────┘
                                   ▼
                         Vallari Storefront
```

This setup allowed me to develop the storefront locally while using Shopify as the commerce platform and content source, then move the completed custom theme through an unpublished development theme before publishing it as the live storefront.