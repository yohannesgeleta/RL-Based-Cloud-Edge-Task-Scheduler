# License Scope

## Project Code

The original contributions to RL-Based Cloud-Edge Task Scheduler are licensed
under the GNU General Public License, version 3 only (`GPL-3.0-only`). A copy of
the license is provided in [LICENSE](LICENSE).

This grant covers the original project code and modifications in the Java
application and in `RLPython/`, including both standalone training and code
intended for Java-dependent training or inference. Choosing the same license
for these components does not assert that they are legally one combined work.

This program is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE. See the GNU General Public License for details.

This grant applies only to contributions whose copyright holders have
authorized distribution under these terms. It does not replace another
copyright holder's license or establish authorship of unidentified code.

## Inherited EdgeCloudSim Code

Much of `src/main/java/edu/boun/edgecloudsim/` comes from
[EdgeCloudSim](https://github.com/CagataySonmez/EdgeCloudSim). Existing copyright
notices attribute that framework to Bogazici University. Preserve those notices
and the applicable upstream GPL terms, including any version permissions in
the original notices. The `GPL-3.0-only` declaration for original project
contributions does not narrow or replace the license of inherited code.

The upstream v4.0 release contains the GPL version 3 text used for this
repository's `LICENSE`. See [PROVENANCE.md](PROVENANCE.md) for the matching
baseline, modified files, and remaining provenance questions.

## Third-Party Dependencies

Libraries under `lib/`, dependencies declared in `pom.xml`, and Python
dependencies retain their own licenses. They are not relicensed by this
repository's GPL declaration. All six checked-in JARs match the corresponding
files in EdgeCloudSim v4.0, but that establishes origin, not completion of their
individual license and source-distribution requirements.

Apache Commons Math includes `META-INF/LICENSE.txt` and `META-INF/NOTICE.txt`
inside its JAR. The other five JARs have no entries with `LICENSE`, `COPYING`,
or `NOTICE` in their names. Their authoritative license notices and any
corresponding-source requirements still need to be collected and recorded.

## Models and Generated Data

No separate license for `.pt` model checkpoints or generated experience data
is established by this document. Their creators, training inputs, applicable
third-party restrictions, and intended redistribution terms must be recorded
before an explicit artifact license can be assigned. The code license alone
does not establish those facts.

## Contributor Rights

Contributors retain copyright in their contributions. Confirm that all
contributors have authorized the applicable project license, and document
any copied code and its original terms. A Git commit identifies a recorded
change; it is not, by itself, proof of copyright ownership or permission to
relicense someone else's work.

## Publishing Checklist

- Include `LICENSE`, this scope document, and `PROVENANCE.md` in distributions.
- Preserve inherited copyright, license, and warranty notices.
- Keep prominent descriptions and dates of modifications to inherited code.
- Confirm contributor rights and resolve unidentified copied code.
- Record dependency licenses, notices, and any required corresponding source
  when distributing binaries.
- Document checkpoint and data provenance separately from source code.

These documents clarify the selected license and the evidence currently
available. They do not certify that every third-party redistribution
requirement has been satisfied.
