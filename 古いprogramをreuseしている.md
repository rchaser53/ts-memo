Create a new 'Program' instance. A Program is an immutable collection of 'SourceFile's and a 'CompilerOptions' that represent a compilation unit.
Old program
reusing

なんか良く分からんがreuseしている
reuseしていないケースの場合(初期の処理?)はundefinedを渡す？

it("works with updated SourceFiles", () => {
    // adapted repro from https://github.com/Microsoft/TypeScript/issues/26166
    const files = [
        { name: "/a.ts", text: SourceText.New("", "", 'import * as a from "a";a;') },
        { name: "/types/zzz/index.d.ts", text: SourceText.New("", "", 'declare module "a" { }') },
    ];
    const host = createTestCompilerHost(files, target);
    const options: CompilerOptions = { target, typeRoots: ["/types"] };
    const program1 = createProgram(["/a.ts"], options, host);
    let sourceFile = program1.getSourceFile("/a.ts")!;
    assert.isDefined(sourceFile, "'/a.ts' is included in the program");
    sourceFile = updateSourceFile(sourceFile, "'use strict';" + sourceFile.text, { newLength: "'use strict';".length, span: { start: 0, length: 0 } });
    assert.strictEqual(sourceFile.statements[2].getSourceFile(), sourceFile, "parent pointers are updated");
    const updateHost: TestCompilerHost = {
        ...host,
        getSourceFile(fileName) {
            return fileName === sourceFile.fileName ? sourceFile : program1.getSourceFile(fileName);
        }
    };
    const program2 = createProgram(["/a.ts"], options, updateHost, program1);
    assert.isDefined(program2.getSourceFile("/a.ts")!.resolvedModules!.get("a"), "'a' is not an unresolved module after re-use");
    assert.strictEqual(sourceFile.statements[2].getSourceFile(), sourceFile, "parent pointers are not altered");
});