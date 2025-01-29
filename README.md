# Using Shadcn UI with Docusaurus

## Part 1: Getting Tailwind CSS working

1. Install Docusaurus

The first thing we have to do is install Docusaurus.

Here is the URL:
https://docusaurus.io/docs/installation

```
npx create-docusaurus@latest shadcn-ui-docusaurus classic --typescript
```

Run the following to see the app working:

```
npm run start
```

1. Set up to use Tailwind CSS

Now we need to make sure to install Tailwind CSS in a way that does not conflict with Docusaurus styles.

```
npm install -D tailwindcss@3.3.5 postcss@8.4.31 autoprefixer@10.4.16
```

Now create tailwind.config.js file in the root of the project.

```
npx tailwindcss init
```

Now, let's modify the tailwind.config.js file to the following:

```
const { fontFamily } = require("tailwindcss/defaultTheme")

/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ["class"],
  content: [
    "./pages/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
    "./src/**/*.{ts,tsx}",
    "./docs/**/*.{js,jsx,ts,tsx,md,mdx}",
    "./blog/**/*.{js,jsx,ts,tsx,md,mdx}",
  ],
  prefix: "",
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      fontFamily: {
        sans: ['"Inter"', ...fontFamily.sans],
        jakarta: ['"Plus Jakarta Sans"', ...fontFamily.sans],
        mono: ['"Fira Code"', ...fontFamily.mono],
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
};
```

And install tailwindcss-animate plugin

```
npm install tailwindcss-animate
```

Now, lets create a `plugins` folder in the root of the project and create a `tailwind-config.cjs` file in it and add this script to it:

```
function tailwindPlugin(context, options) {
  return {
    name: 'tailwind-plugin',
    configurePostCss(postcssOptions) {
      postcssOptions.plugins = [
        require('postcss-import'),
        require('tailwindcss'),
        require('autoprefixer'),
      ];
      return postcssOptions;
    },
  };
}

module.exports = tailwindPlugin;
```

Let’s come back to our docusaurus.config.js file and add the following

```
import tailwindPlugin from "./plugins/tailwind-config.cjs"; // add this
.
.
.
.
const config = {
.
.
.
plugins: [tailwindPlugin], // update this
.
```

Now, lets update the `src/css/custom.css` file to add this to the top of the file:

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

At this point, we should be able to run `npm run start` and see that the app is working and the default Docusaurus styles are respected.

Now we should be able to test that we can use tailwind classes in our components by making a small change to `src/components/HomepageFeatures/index.tsx` and adding a `className` prop to the `div` element.

For example, we can change:

```
function Feature({title, Svg, description}: FeatureItem) {
  return (
    <div className={clsx('col col--4')}>
      <div className="text--center">
        <Svg className={styles.featureSvg} role="img" />
      </div>
      <div className="text--center padding-horiz--md">
        <Heading as="h3">{title}</Heading>
        <p>{description}</p>
      </div>
    </div>
  );
}
```

to

```
function Feature({title, Svg, description}: FeatureItem) {
  return (
    <div className={clsx('col col--4')}>
      <div className="text--center bg-red-500">
        <Svg className={styles.featureSvg} role="img" />
      </div>
      <div className="text--center padding-horiz--md">
        <Heading as="h3">{title}</Heading>
        <p>{description}</p>
      </div>
    </div>
  );
}
```

We can see that the `div` element now has a red background color.

Cool, we could use the power of tailwindcss inside docusaurus without destroying the default styles by docusaurus.

## Part 2: Adding Shadcn UI to the project