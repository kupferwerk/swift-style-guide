# SwiftLint

[SwiftLint](https://github.com/realm/SwiftLint#disable-rules-in-code) is a great way to enforce a common style for Swift source code. The supplied configuration file in this repository is meant to be a starting point for all project. Some projects might need adjustments like excluding of some rules or not applying some rules for certain files.

## CocoaPod integration
If your project is using CocoaPods as a dependency management tool you can easily integrate SwiftLint into your project without needing to manually install SwiftLint onto each dev machine and your CI server.

1. Add `pod 'SwiftLint'` to your Podfile
2. Add a run script build phase with following content:

	```bash
	"${PODS_ROOT}/SwiftLint/swiftlint" --path ${SRCROOT}/${EXECUTABLE_NAME} --config ../.swiftlint.yml
	```
	(Assuming you want to lint the content of the folder named after your project name and having the linting file inside the root of your project.)

3. Don't forget to enable swiftlint for each target separately.

## Temporary disable rules
Sometimes you need to silence a linter warning just for a single line/method. You can do this like it is described [here](https://github.com/realm/SwiftLint#disable-rules-in-code).