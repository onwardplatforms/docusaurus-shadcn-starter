# Using Shadcn UI with Docusaurus

The goals of this guide is to:

Make it so you can use Shadcn UI components in Docusaurus out of the box. This includes:
- Enabling tailwindcss without impacting the default Docusaurus styles
- Being able to install them with the CLI and have them show in in the right place (src/components/ui)
- Use them without any modifications (such as Tailwind prefixes or imports of the @/lib/utils file)

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
  		padding: '2rem',
  		screens: {
  			'2xl': '1400px'
  		}
  	},
  	extend: {
  		fontFamily: {
  			sans: [
  				'Inter"',
                    ...fontFamily.sans
                ],
  			jakarta: [
  				'Plus Jakarta Sans"',
                    ...fontFamily.sans
                ],
  			mono: [
  				'Fira Code"',
                    ...fontFamily.mono
                ]
  		},
  		borderRadius: {
  			lg: 'var(--radius)',
  			md: 'calc(var(--radius) - 2px)',
  			sm: 'calc(var(--radius) - 4px)'
  		},
  		colors: {
  			border: 'hsl(var(--border))',
  			input: 'hsl(var(--input))',
  			ring: 'hsl(var(--ring))',
  			background: 'hsl(var(--background))',
  			foreground: 'hsl(var(--foreground))',
  			primary: {
  				DEFAULT: 'hsl(var(--primary))',
  				foreground: 'hsl(var(--primary-foreground))'
  			},
  			secondary: {
  				DEFAULT: 'hsl(var(--secondary))',
  				foreground: 'hsl(var(--secondary-foreground))'
  			},
  			destructive: {
  				DEFAULT: 'hsl(var(--destructive))',
  				foreground: 'hsl(var(--destructive-foreground))'
  			},
  			muted: {
  				DEFAULT: 'hsl(var(--muted))',
  				foreground: 'hsl(var(--muted-foreground))'
  			},
  			accent: {
  				DEFAULT: 'hsl(var(--accent))',
  				foreground: 'hsl(var(--accent-foreground))'
  			},
  			popover: {
  				DEFAULT: 'hsl(var(--popover))',
  				foreground: 'hsl(var(--popover-foreground))'
  			},
  			card: {
  				DEFAULT: 'hsl(var(--card))',
  				foreground: 'hsl(var(--card-foreground))'
  			},
  			sidebar: {
  				DEFAULT: 'hsl(var(--sidebar-background))',
  				foreground: 'hsl(var(--sidebar-foreground))',
  				primary: 'hsl(var(--sidebar-primary))',
  				'primary-foreground': 'hsl(var(--sidebar-primary-foreground))',
  				accent: 'hsl(var(--sidebar-accent))',
  				'accent-foreground': 'hsl(var(--sidebar-accent-foreground))',
  				border: 'hsl(var(--sidebar-border))',
  				ring: 'hsl(var(--sidebar-ring))'
  			}
  		},
  		keyframes: {
  			'accordion-down': {
  				from: {
  					height: '0'
  				},
  				to: {
  					height: 'var(--radix-accordion-content-height)'
  				}
  			},
  			'accordion-up': {
  				from: {
  					height: 'var(--radix-accordion-content-height)'
  				},
  				to: {
  					height: '0'
  				}
  			}
  		},
  		animation: {
  			'accordion-down': 'accordion-down 0.2s ease-out',
  			'accordion-up': 'accordion-up 0.2s ease-out'
  		}
  	}
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
  // Plugins
  plugins: [
    tailwindPlugin,
    function (context, options) {
      return {
        name: 'docusaurus-plugin-aliases',
        configureWebpack() {
          return {
            resolve: {
              alias: {
                '@': path.resolve(__dirname, 'src'),
              },
            },
          };
        },
      };
    },
  ],
.
```

The reason moving to the plugin format worked is because of Docusaurus's type system and architecture:
In the Config type (which we're using with const config: Config), configureWebpack is not a valid top-level property. The TypeScript error was telling us this.
2. However, Docusaurus's plugin system is specifically designed to handle webpack configurations. Plugins can implement a configureWebpack method as part of their lifecycle hooks. This is part of the official plugin API.
The structure:
function (context, options) {
}
follows Docusaurus's plugin specification, where plugins can modify webpack configuration through their lifecycle hooks rather than through top-level config properties.
This is more modular and follows Docusaurus's architectural pattern of using plugins to extend functionality, rather than putting everything in the main config.

Now, lets update the `src/css/custom.css` file to add shadcn styles:

```
/**
 * Any CSS included here will be global. The classic template
 * bundles Infima by default. Infima is a CSS framework designed to
 * work well for content-centric websites.
 */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }

  [data-theme='dark'] {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

/* You can override the default Infima variables here. */
:root {
  --ifm-color-primary: #2e8555;
  --ifm-color-primary-dark: #29784c;
  --ifm-color-primary-darker: #277148;
  --ifm-color-primary-darkest: #205d3b;
  --ifm-color-primary-light: #33925d;
  --ifm-color-primary-lighter: #359962;
  --ifm-color-primary-lightest: #3cad6e;
  --ifm-code-font-size: 95%;
  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.1);

  /* Shadcn UI variables with sh- prefix */
  --sh-background: 0 0% 100%;
  --sh-foreground: 240 10% 3.9%;
  --sh-card: 0 0% 100%;
  --sh-card-foreground: 240 10% 3.9%;
  --sh-popover: 0 0% 100%;
  --sh-popover-foreground: 240 10% 3.9%;
  --sh-primary: 217.2 91.2% 59.8%;
  --sh-primary-foreground: 0 0% 98%;
  --sh-secondary: 240 4.8% 95.9%;
  --sh-secondary-foreground: 240 5.9% 10%;
  --sh-muted: 240 4.8% 95.9%;
  --sh-muted-foreground: 240 3.8% 46.1%;
  --sh-accent: 240 4.8% 95.9%;
  --sh-accent-foreground: 240 5.9% 10%;
  --sh-destructive: 0 84.2% 60.2%;
  --sh-destructive-foreground: 0 0% 98%;
  --sh-border: 240 5.9% 90%;
  --sh-input: 240 5.9% 90%;
  --sh-ring: 217.2 91.2% 59.8%;
  --radius: 0.5rem;
}

/* For readability concerns, you should choose a lighter palette in dark mode. */
[data-theme='dark'] {
  --ifm-color-primary: #25c2a0;
  --ifm-color-primary-dark: #21af90;
  --ifm-color-primary-darker: #1fa588;
  --ifm-color-primary-darkest: #1a8870;
  --ifm-color-primary-light: #29d5b0;
  --ifm-color-primary-lighter: #32d8b4;
  --ifm-color-primary-lightest: #4fddbf;
  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.3);

  /* Shadcn UI dark mode variables with sh- prefix */
  --sh-background: 240 10% 3.9%;
  --sh-foreground: 0 0% 98%;
  --sh-card: 240 10% 3.9%;
  --sh-card-foreground: 0 0% 98%;
  --sh-popover: 240 10% 3.9%;
  --sh-popover-foreground: 0 0% 98%;
  --sh-primary: 217.2 91.2% 59.8%;
  --sh-primary-foreground: 0 0% 98%;
  --sh-secondary: 240 3.7% 15.9%;
  --sh-secondary-foreground: 0 0% 98%;
  --sh-muted: 240 3.7% 15.9%;
  --sh-muted-foreground: 240 5% 64.9%;
  --sh-accent: 240 3.7% 15.9%;
  --sh-accent-foreground: 0 0% 98%;
  --sh-destructive: 0 62.8% 30.6%;
  --sh-destructive-foreground: 0 0% 98%;
  --sh-border: 240 3.7% 15.9%;
  --sh-input: 240 3.7% 15.9%;
  --sh-ring: 217.2 91.2% 59.8%;
}
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

Add the `lib/utils.ts` file to the project:

First install dependencies:
```
npm install clsx tailwind-merge
```

Now add the `lib/utils.ts` file to the project:

```
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
    return twMerge(clsx(inputs))
} 
```

update the tsconfig.json file to the following to match shadcn preference:

```
{
  // This file is not used in compilation. It is here just for a nice editor experience.
  "extends": "@docusaurus/tsconfig",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      // This is for compatability with Docusaurus
      "@site/*": [
        "./*"
      ],
      // This is for compatability with Shadcn UI
      "@/*": [
        "./src/*"
      ]
    }
  },
  "exclude": [
    ".docusaurus",
    "build"
  ]
}
```

Add the `components.json` file to the project:

```
{
    "$schema": "https://ui.shadcn.com/schema.json",
    "style": "new-york",
    "rsc": false,
    "tsx": true,
    "tailwind": {
        "config": "tailwind.config.js",
        "css": "src/index.css",
        "baseColor": "zinc",
        "cssVariables": true,
        "prefix": ""
    },
    "aliases": {
        "components": "@/components",
        "utils": "@/lib/utils",
        "ui": "@/components/ui",
        "lib": "@/lib",
        "hooks": "@/hooks"
    },
    "iconLibrary": "lucide"
}
```

Install shadcn ui components with the cli as normal 

```
npx shadcn@latest add button
```

You should now see the button component in the project.

Finally, lets test it out by adding a button to the `src/components/HomepageFeatures/index.tsx` file.

```
function Feature({title, Svg, description}: FeatureItem) {
  return (
    <div className={clsx('col col--4')}>
      <div className="text--center">
        <Button>Click me</Button>
      </div>
    </div>
  );
}
```

Note:

When we rewrite this, just have a single command to install all depenedencies:

```
npm add tailwindcss-animate class-variance-authority clsx tailwind-merge lucide-react radix-ui/react-icons
```