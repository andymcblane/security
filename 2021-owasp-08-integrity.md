# Integrity
_Imagine one of the GitHub actions you use is comprimised_. They manange to push a malicious version to an existing tag `v4` that you had pinned.

The code is downloaded and executed across all organisation runners on the Monday morning.

It looks like tools like Aikido can address enforcement of this, possibly in combination with custom CI scripts.