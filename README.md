# Trusted Checksums Demo

This little project is a demo for "Trusted Checksums" feature of Maven 3.9.x.

Note: The `.mvn/maven.config` will intentionally set your local repository to
`./local`, just to not mess with your local repository, but also to force
full download (as post checkout, this directory is not present).

After you checked out, you can try to build it:
```
$ mvn clean install
```

The demo project will download everything it needs, and install the project.
What is not quite visible: it used provided SHA-512 to validate the downloaded
artifacts.

Note: the demo uses SHA-512 despite Maven Central does not provide those.
The trick is that "we provide" those trusted checksums, that are committed
into git repository along with out (trusted) sources.

If you inspect your local repository, you will find `.sha512` checksums
instead of the usual `.sha1` ones. Everything happens "as before", except
we provided these trusted checksums ahead of actual HTTP transport to 
resolver, and resolver used those to validate transported artifacts.

The trusted checksums are looked up from 
`.mvn/checksums/checksums-${repository.id}.${checksumExtension}` file.

## Exercise 1: change any dependency

Try to change the version of some dependency in POM, and rebuild.

It will fail, as "fail if missing" is enabled: if checksum not found,
the build will fail (as resolution will be prevented).

## Exercise 2: add new dependencies

Try to un-comment the junit:junit test dependency. It will fail as well
just like in Exercise 1.

If you want to "add" dependencies, enable "recording":

```
mvn clean install -Daether.artifactResolver.postProcessor.trustedChecksums.record
```

At the build end, you will see following message:
```
[INFO] Saving 213 checksums to '/home/cstamas/tmp/tc-demo/.mvn/checksums/checksums-central.sha512'
```

In "record" mode post processor instead of "enforcing" it listens and calculates checksums
and saves them to file at session end.

Naturally, you can manually edit the `checksums-central.sha512`, or, if you noticed, the
file format is compatible to that of GNU Binutils, so you can even use those tools
to produce the files.

