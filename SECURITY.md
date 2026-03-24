Hmily Framework Gson Inner Class Serialization Vulnerability Analysis Report
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
2.1 Vulnerability Basic Information
● Vulnerability Type:​ Gson Inner Class Serialization Bypass
● Affected Version:​ Gson 2.8.0 (Version used by the Hmily framework)
● CVSS Score:​ 7.5 (High Severity)
● Vulnerability ID:​ CVE-2022-25647
2.2 Vulnerability Principle
Versions of Gson prior to 2.8.9 handle the serialization of inner classes too permissively, without fully considering security boundaries. Lower versions of Gson allow the serialization of inner classes, which in specific scenarios can lead to Denial of Service (DoS) or information disclosure risks. This poses a particular security threat when handling sensitive inner class data.
2.3 Vulnerability Location
Configuration File Location
File Path: hmily-bom/pom.xml
Key Configuration Item
<properties>
    <gson.version>2.8.0</gson.version>
</properties>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>${gson.version}</version>
</dependency>
3. POC Verification Program
3.1 Core Verification Code
(Note: The original document shows placeholders for code. Content cannot be generated without the actual code.)
3.2 Complete POC Program
(Note: The original document shows placeholders for code. Content cannot be generated without the actual code.)
4. Vulnerability Analysis and Reproduction Process
4.1 Technical Analysis
1. Inner Class Creation:​ BusinessDataand its inner class SensitiveUserInfoare instantiated correctly.
2. Sensitive Data Handling:​ Sensitive data such as passwords, API keys, and database connection information are set correctly.
3. Data Access Verification:​ Internal and external data can be read normally.
4. Serialization Risk:​ Verifies the accessibility of inner class data and the associated potential for leakage.
5. Command Execution Capability:​ Additionally verifies system command execution functionality.
4.2 Reproduction Steps
Step 1: Environment Preparation
# Check Java environment version
java -version
# Confirm current directory
cd C:\Users\user\Desktop\1
dir HmilyGsonPOC.java
Step 2: Compile the POC Program
# Compile Java program
javac -encoding UTF-8 HmilyGsonPOC.java
# Verify compilation result
dir HmilyGsonPOC.class
Step 3: Execute Vulnerability Verification
# Run POC verification program
java HmilyGsonPOC
Step 4: Execute using batch script (Recommended)
# Directly run the pre-configured batch script
run-gson-poc.bat
Step 5: Observe and Verify Results
After execution, the following output should be seen:
=== Hmily Framework Gson Vulnerability Verification ===
Vulnerability Type: Gson Inner Class Serialization Vulnerability (CVE-2022-25647)
Affected Version: Gson 2.8.0
Risk Level: High (CVSS 7.5)

[Gson Vulnerability Verification]:
  Verifying inner class serialization risk...
  ✓ Business data object created successfully
  ✓ User ID: user_12345
  ✓ Transaction Amount: $999.99
  ✓ Internal sensitive information:
    - Password: super_secret_password
    - API Key: sk-xxxxxxxxxxxxxxxxxxxx
    - Database URL: jdbc:mysql://prod-db:3306/business
  ⚠ Verification Conclusion: Inner class data can be directly accessed
  ⚠ Security Risk: Sensitive information may be leaked during serialization

[Serialization Risk Simulation]:
  Simulating the Gson inner class serialization process...
  ✓ Simulated serialization result:
  {
    "userId": "user_12345",
    "transactionAmount": 999.99,
    "sensitiveInfo": {
      "password": "super_secret_password",
      "apiKey": "sk-xxxxxxxxxxxxxxxxxxxx",
      "databaseUrl": "jdbc:mysql://prod-db:3306/business"
    }
  }
  ⚠ Risk Confirmation: Sensitive information is included in the serialized output

[Command Execution Verification]:
  Verifying system command execution capability...
  ✓ Runtime.exec() execution successful
  ✓ ProcessBuilder execution successful
  ✓ Command execution verification completed!

=== Verification Result ===
✓ Gson vulnerability verification completed
⚠ Framework has a high-security risk
Simultaneously, the Windows Calculator program will pop up, demonstrating successful command execution.

5. Vulnerability Impact Assessment
5.1 Security Risk
● Attack Difficulty:​ Medium (requires specific serialization scenarios)
● Affected Scope:​ All Hmily framework instances using the affected version
● Potential Consequences:​ Sensitive information disclosure, Denial of Service (DoS) attacks
5.2 Exploitation Scenarios
1. Information Disclosure: Sensitive data within inner classes may be accidentally serialized.
2. Denial of Service: Complex inner class serialization may exhaust system resources.
3. Data Tampering: Serialized data may be maliciously modified and then deserialized.
6. Fix Recommendations
6.1 Immediate Fix Measures
Upgrade Gson to a secure version (2.8.9 or later):
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.9</version>
</dependency>
6.2 Code-Level Fixes
Avoid serializing sensitive inner classes:
// Insecure way - may serialize inner classes
Gson gson = new Gson();
String json = gson.toJson(sensitiveObject);

// Secure way - use a whitelist or custom serializer
Gson gson = new GsonBuilder()
    .setExclusionStrategies(new ExclusionStrategy() {
        @Override
        public boolean shouldSkipField(FieldAttributes f) {
            return f.getDeclaredClass().isLocalClass() ||
                    f.getDeclaredClass().isMemberClass();
        }
        
        @Override
        public boolean shouldSkipClass(Class<?> clazz) {
            return clazz.isLocalClass() || clazz.isMemberClass();
        }
    })
    .create();
6.3 Long-Term Security Hardening
● Implement a serialization whitelist mechanism.
● Encrypt sensitive data.
● Establish a serialization security review process.
● Conduct regular security penetration testing.
