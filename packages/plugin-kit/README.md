# ESLint Plugin Kit

## Description

A collection of utilities to help build ESLint plugins.

## Installation

For Node.js and compatible runtimes:

```shell
npm install @eslint/plugin-kit
# or
yarn add @eslint/plugin-kit
# or
pnpm install @eslint/plugin-kit
# or
bun install @eslint/plugin-kit
```

For Deno:

```shell
deno add @eslint/plugin-kit
```

## Usage

This package exports the following utilities:

-   `ConfigCommentParser` - used to parse ESLint configuration comments (i.e., `/* eslint-disable rule */`)
-   `VisitNodeStep` and `CallMethodStep` - used to help implement `SourceCode#traverse()`
-   `TextSourceCodeBase` - base class to help implement the `SourceCode` interface

### `ConfigCommentParser`

To use the `ConfigCommentParser` class, import it from the package and create a new instance, such as:

```js
import { ConfigCommentParser } from "@eslint/plugin-kit";

// create a new instance
const commentParser = new ConfigCommentParser();

// pass in a comment string without the comment delimiters
const directive = commentParser.parseDirective(
	"eslint-disable prefer-const, semi -- I don't want to use these.",
);

// will be undefined when a directive can't be parsed
if (directive) {
	console.log(directive.label); // "eslint-disable"
	console.log(directive.value); // "prefer-const, semi"
	console.log(directive.justification); // "I don't want to use these"
}
```

There are different styles of directive values that you'll need to parse separately to get the correct format:

```js
import { ConfigCommentParser } from "@eslint/plugin-kit";

// create a new instance
const commentParser = new ConfigCommentParser();

// list format
const list = commentParser.parseListConfig("prefer-const, semi");
console.log(Object.entries(list)); // [["prefer-const", true], ["semi", true]]

// string format
const strings = commentParser.parseStringConfig("foo:off, bar");
console.log(Object.entries(strings)); // [["foo", "off"], ["bar", null]]

// JSON-like config format
const jsonLike = commentParser.parseJSONLikeConfig(
	"semi:[error, never], prefer-const: warn",
);
console.log(Object.entries(jsonLike.config)); // [["semi", ["error", "never"]], ["prefer-const", "warn"]]
```

### `VisitNodeStep` and `CallMethodStep`

The `VisitNodeStep` and `CallMethodStep` classes represent steps in the traversal of source code. They implement the correct interfaces to return from the `SourceCode#traverse()` method.

The `VisitNodeStep` class is the more common of the two, where you are describing a visit to a particular node during the traversal. The constructor accepts three arguments:

-   `target` - the node being visited. This is used to determine the method to call inside of a rule. For instance, if the node's type is `Literal` then ESLint will call a method named `Literal()` on the rule (if present).
-   `phase` - either 1 for enter or 2 for exit.
-   `args` - an array of arguments to pass into the visitor method of a rule.

For example:

```js
import { VisitNodeStep } from "@eslint/plugin-kit";

class MySourceCode {
	traverse() {
		const steps = [];

		for (const { node, parent, phase } of iterator(this.ast)) {
			steps.push(
				new VisitNodeStep({
					target: node,
					phase: phase === "enter" ? 1 : 2,
					args: [node, parent],
				}),
			);
		}

		return steps;
	}
}
```

The `CallMethodStep` class is less common and is used to tell ESLint to call a specific method on the rule. The constructor accepts two arguments:

-   `target` - the name of the method to call, frequently beginning with `"on"` such as `"onCodePathStart"`.
-   `args` - an array of arguments to pass to the method.

For example:

```js
import { VisitNodeStep, CallMethodStep } from "@eslint/plugin-kit";

class MySourceCode {

    traverse() {

        const steps = [];

        for (const { node, parent, phase } of iterator(this.ast)) {
            steps.push(
                new VisitNodeStep({
                    target: node,
                    phase: phase === "enter" ? 1 : 2,
                    args: [node, parent],
                }),
            );

            // call a method indicating how many times we've been through the loop
            steps.push(
                new CallMethodStep({
                    target: "onIteration",
                    args: [steps.length]
                });
            )
        }

        return steps;
    }
}
```

### `TextSourceCodeBase`

The `TextSourceCodeBase` class is intended to be a base class that has several of the common members found in `SourceCode` objects already implemented. Those members are:

-   `lines` - an array of text lines that is created automatically when the constructor is called.
-   `getLoc(node)` - gets the location of a node. Works for nodes that have the ESLint-style `loc` property and nodes that have the Unist-style [`position` property](https://github.com/syntax-tree/unist?tab=readme-ov-file#position). If you're using an AST with a different location format, you'll still need to implement this method yourself.
-   `getRange(node)` - gets the range of a node within the source text. Works for nodes that have the ESLint-style `range` property and nodes that have the Unist-style [`position` property](https://github.com/syntax-tree/unist?tab=readme-ov-file#position). If you're using an AST with a different range format, you'll still need to implement this method yourself.
-   `getText(nodeOrToken, charsBefore, charsAfter)` - gets the source text for the given node or token that has range information attached. Optionally, can return additional characters before and after the given node or token. As long as `getRange()` is properly implemented, this method will just work.
-   `getAncestors(node)` - returns the ancestry of the node. In order for this to work, you must implement the `getParent()` method yourself.

Here's an example:

```js
import { TextSourceCodeBase } from "@eslint/plugin-kit";

export class MySourceCode extends TextSourceCodeBase {
	#parents = new Map();

	constructor({ ast, text }) {
		super({ ast, text });
	}

	getParent(node) {
		return this.#parents.get(node);
	}

	traverse() {
		const steps = [];

		for (const { node, parent, phase } of iterator(this.ast)) {
			//save the parent information
			this.#parent.set(node, parent);

			steps.push(
				new VisitNodeStep({
					target: node,
					phase: phase === "enter" ? 1 : 2,
					args: [node, parent],
				}),
			);
		}

		return steps;
	}
}
```

In general, it's safe to collect the parent information during the `traverse()` method as `getParent()` and `getAncestor()` will only be called from rules once the AST has been traversed at least once.

## License

Apache 2.0

## Sponsors

The following companies, organizations, and individuals support ESLint's ongoing maintenance and development. [Become a Sponsor](https://eslint.org/donate) to get your logo on our README and website.

<!-- NOTE: This section is autogenerated. Do not manually edit.-->
<!--sponsorsstart-->
<h3>Platinum Sponsors</h3>
<p><a href="https://automattic.com"><img src="https://images.opencollective.com/automattic/d0ef3e1/logo.png" alt="Automattic" height="128"></a> <a href="https://www.airbnb.com/"><img src="https://images.opencollective.com/airbnb/d327d66/logo.png" alt="Airbnb" height="128"></a></p><h3>Gold Sponsors</h3>
<p><a href="https://trunk.io/"><img src="https://images.opencollective.com/trunkio/avatar.png" alt="trunk.io" height="96"></a> <a href="https://opensource.siemens.com"><img src="https://avatars.githubusercontent.com/u/624020?v=4" alt="Siemens" height="96"></a></p><h3>Silver Sponsors</h3>
<p><a href="https://www.jetbrains.com/"><img src="https://images.opencollective.com/jetbrains/fe76f99/logo.png" alt="JetBrains" height="64"></a> <a href="https://liftoff.io/"><img src="https://images.opencollective.com/liftoff/5c4fa84/logo.png" alt="Liftoff" height="64"></a> <a href="https://americanexpress.io"><img src="https://avatars.githubusercontent.com/u/3853301?v=4" alt="American Express" height="64"></a> <a href="https://www.workleap.com"><img src="https://avatars.githubusercontent.com/u/53535748?u=d1e55d7661d724bf2281c1bfd33cb8f99fe2465f&v=4" alt="Workleap" height="64"></a></p><h3>Bronze Sponsors</h3>
<p><a href="https://www.notion.so"><img src="https://images.opencollective.com/notion/bf3b117/logo.png" alt="notion" height="32"></a> <a href="https://www.crosswordsolver.org/anagram-solver/"><img src="https://images.opencollective.com/anagram-solver/2666271/logo.png" alt="Anagram Solver" height="32"></a> <a href="https://icons8.com/"><img src="https://images.opencollective.com/icons8/7fa1641/logo.png" alt="Icons8" height="32"></a> <a href="https://discord.com"><img src="https://images.opencollective.com/discordapp/f9645d9/logo.png" alt="Discord" height="32"></a> <a href="https://www.ignitionapp.com"><img src="https://avatars.githubusercontent.com/u/5753491?v=4" alt="Ignition" height="32"></a> <a href="https://nx.dev"><img src="https://avatars.githubusercontent.com/u/23692104?v=4" alt="Nx" height="32"></a> <a href="https://herocoders.com"><img src="https://avatars.githubusercontent.com/u/37549774?v=4" alt="HeroCoders" height="32"></a> <a href="https://usenextbase.com"><img src="https://avatars.githubusercontent.com/u/145838380?v=4" alt="Nextbase Starter Kit" height="32"></a></p>
<!--sponsorsend-->

<!--techsponsorsstart-->
<h2>Technology Sponsors</h2>
<p><a href="https://netlify.com"><img src="https://raw.githubusercontent.com/eslint/eslint.org/main/src/assets/images/techsponsors/netlify-icon.svg" alt="Netlify" height="32"></a> <a href="https://algolia.com"><img src="https://raw.githubusercontent.com/eslint/eslint.org/main/src/assets/images/techsponsors/algolia-icon.svg" alt="Algolia" height="32"></a> <a href="https://1password.com"><img src="https://raw.githubusercontent.com/eslint/eslint.org/main/src/assets/images/techsponsors/1password-icon.svg" alt="1Password" height="32"></a>
</p>
<!--techsponsorsend-->