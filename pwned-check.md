---
layout: page
homepage: true
short_title: "Module: Pwned Check"
side_menu: true
permalink: /pwned-check/
---

# Suspicious Credentials Check

This module provides functionality of checking if the credentials being used for login attempt have been compromised in a data breach, 
in other words '_pwned_', hence the module's name.

The credentials typed by the user are transformed into hashes and checked against presence in a local database containing hashes of 
credentials that have been leaked in a known data breach. 

---
> **The check is done completely locally, no data is sent outside your application.**
>
> **Since it is performed on hashes of credentials, not original values, the risk of a password leak is not increased by using this feature.**

---

## How Does It Work

The functionality operates on hashes of credentials and uses [Bloom filter](https://en.wikipedia.org/wiki/Bloom_filter) for doing the check. 

The Bloom filter implementation being used offers great performance and can be applied on hot paths, 
you can read [detailed documentation here]({{ site.baseurl }}/additional-features/#file-based-bloom-filter-for-java) 
or see [the source code here](https://github.com/nixer-io/nixer-spring-plugin/tree/master/bloom-filter).

## Usage
### Data Source

Before using the filter it must be populated with leaked credentials data to be checked against. 
For this purpose we provide convenient command line utility, 
[bloom-tool]({{ site.baseurl }}/additional-features/#bloom-cli-tool), which allows generating, manipulating 
and testing Bloom filters.

As a data source you can use pwned passwords lists available at [haveibeenpwned.com](https://haveibeenpwned.com/Passwords) 
or similar websites, or can be obtained from any credentials breach data you have access to. 
 
For information about transforming the data into Bloom filter please refer to 
[bloom-tool documentation](https://github.com/nixer-io/nixer-spring-plugin/tree/master/bloom-tool).

Bloom filter is represented by two files:
- `filter_name.bloom` - metadata file containing filter parameters, 
- `filter_name.bloom-data` - data set file.

### Installation

Pwned Check Nixer plugin is distributed through [Maven Central]({{ site.project.mvn_repo_url }}).
It requires dependency to Core Nixer plugin as well.

```kotlin
dependencies {
    implementation("io.nixer:nixer-plugin-core:{{ site.data.nixer_version.latest_release }}")
    implementation("io.nixer:nixer-plugin-pwned-check:{{ site.data.nixer_version.latest_release }}")
}
```

### Configuration

In order to enable the functionality the following properties are to be set:

```properties
nixer.pwned.check.enabled=true
nixer.pwned.check.pwnedFilePath=classpath:PWNED_DATABASE_DIRECTORY/filter_name.bloom
```

where `PWNED_DATABASE_DIRECTORY` is expected to contain both Bloom filter files, `filter_name.bloom` and `filter_name.bloom-data`.

### Results

Pwned check results are written into 
[Spring Boot application metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics), 
under `pwned_check` metric name, from where they can be utilized for any mitigation actions.
