---
title: "Transport of V3C Bitstreams over Media over QUIC Transport (MOQT)"
abbrev: "V3C over MOQT"
category: info
docname: draft-yuyou-moq-v3c-01
ipr: trust200902
submissiontype: IETF
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Media Over QUIC"
keyword:
  - v3c
  - volumetric
  - media over quic
  - moq
  - moqt
  - msf
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: yuyou/v3c
  latest: https://yuyou.github.io/v3c/
author:
  -
    ins: Y. You
    name: Yu You
    organization: Nokia
    email: yu.you@nokia.com
  -
    ins: S. Gül
    name: Serhan Gül
    organization: Nokia
    email: serhan.guel@nokia.com
  -
    ins: L. Kondrad
    name: Lukasz Kondrad
    organization: Nokia
    email: lukasz.kondrad@nokia.com
  -
    ins: P. Rondao Alface
    name: Patrice Rondao Alface
    organization: Nokia
    email: patrice.rondao_alface@nokia.com
  -
    ins: L. Ilola
    name: Lauri Ilola
    organization: Nokia
    email: lauri.ilola@nokia.com

normative:
  I-D.ietf-moq-loc:
  I-D.ietf-moq-msf:
  I-D.ietf-moq-transport-23:
  ISOIEC14496-15:
    title: "Information technology -- Coding of audio-visual objects -- Part 15: Carriage of network abstraction layer (NAL) unit structured video in the ISO base media file format"
    author:
      - org: ISO/IEC
    seriesinfo:
      ISO/IEC: 14496-15
  ISOIEC23090-5:
    title: "Information technology -- Coded representation of immersive media -- Part 5: Visual volumetric video-based coding (V3C) and video-based point cloud compression (V-PCC)"
    author:
      - org: ISO/IEC
    date: 2026
    seriesinfo:
      ISO/IEC: 23090-5:2026
    target: https://www.iso.org/standard/91546.html
  ISOIEC23090-10:
    title: "Information technology -- Coded representation of immersive media -- Part 10: Carriage of visual volumetric video-based coding data"
    author:
      - org: ISO/IEC
    date: 2026
    seriesinfo:
      ISO/IEC: "23090-10:2026 (DIS)"
  ISOIEC23090-29:
    title: "Information technology -- Coded representation of immersive media -- Part 29: Video-based dynamic mesh coding (V-DMC)"
    author:
      - org: ISO/IEC
    date: 2026
    seriesinfo:
      ISO/IEC: 23090-29:2026
    target: https://www.iso.org/standard/85254.html
  RFC4648:
  RFC9000:
  RFC9221:

informative:
  I-D.ietf-moq-cmsf:
  I-D.ietf-moq-secure-objects:
  I-D.ietf-webtrans-http3:
  I-D.ietf-webtrans-overview:
  ISOIEC14496-12:
    title: "Information technology -- Coding of audio-visual objects -- Part 12: ISO base media file format"
    author:
      - org: ISO/IEC
    seriesinfo:
      ISO/IEC: 14496-12
  ISOIEC23090-12:
    title: "Information technology -- Coded representation of immersive media -- Part 12: MPEG Immersive video (MIV)"
    author:
      - org: ISO/IEC
    date: 2025
    seriesinfo:
      ISO/IEC: 23090-12:2025
    target: https://www.iso.org/standard/87643.html
  RFC10034:
  W3C-WEBCODECS:
    title: "WebCodecs"
    author:
      - org: World Wide Web Consortium
    target: https://www.w3.org/TR/webcodecs/

...

--- abstract

This document specifies a mapping and packaging for transporting
ISO/IEC 23090-5 Visual Volumetric Video-based Coding (V3C) bitstreams
over Media over QUIC Transport (MOQT).  The mapping uses the MOQT
Streaming Format (MSF): V3C configuration is MSF initialization data,
each independently consumable V3C component is an MSF track, a run of
V3C composition units that starts at a random access point is a MOQT
Group, and the coded data of one component for one composition unit
is a MOQT Object.  Relays remain unaware of V3C syntax.  V3C knowledge is
confined to the streaming-format layer.

--- middle

# Introduction

Visual Volumetric Video-based Coding (V3C) as specified in
{{ISOIEC23090-5}} represents volumetric media as multiple synchronized
component bitstreams and associated atlas data.  A V3C bitstream is a
sequence of V3C units.  Unit types defined by {{ISOIEC23090-5}} are
the V3C parameter set (VPS), atlas data (AD), occupancy video data
(OVD), geometry video data (GVD), attribute video data (AVD), packed
video data (PVD), common atlas data (CAD), basemesh data (BMD), and
arithmetic-coded displacement data (ADD).  {{RFC10034}}, Table 1,
summarizes an earlier unit-type list in which types 7 through 31 were
reserved; this document follows {{ISOIEC23090-5}} for the current
identifiers ({{v3c-track-identifiers}}).

V3C defines a generic high-level syntax for representing volumetric
media as a collection of synchronized component bitstreams, together
with signaling for their identification, configuration, and
reconstruction.  {{ISOIEC23090-5}}, Clause 6.3,
defines application extensions that plug into that syntax:
Video-based Point Cloud Compression (V-PCC) in Annex H of
{{ISOIEC23090-5}}, MPEG Immersive Video (MIV) in {{ISOIEC23090-12}},
and Video-based Dynamic Mesh Coding (V-DMC) in {{ISOIEC23090-29}}.
This document maps the generic V3C unit stream to the MOQT data
model.  It does not specify V-PCC reconstruction, MIV camera or
common-atlas payload syntax, or V-DMC mesh decoding.  Atlas and VPS
extensions (`asps_vpcc_extension`, `asps_miv_extension`,
`asps_vdmc_extension`, `vps_extension`) remain inside VPS and atlas
payloads.

Media over QUIC Transport (MOQT) {{I-D.ietf-moq-transport-23}} provides
publish/subscribe-based delivery over QUIC {{RFC9000}} and WebTransport.
The MOQT Streaming Format (MSF) {{I-D.ietf-moq-msf}} defines a catalog,
initialization data, time-aligned groups, and track selection on top
of MOQT.  MSF defines Low Overhead Container (LOC) {{I-D.ietf-moq-loc}}
packaging for audio and video ({{I-D.ietf-moq-msf}}, Section 4.1), in
which each encoded audio or video sample is one MOQT Object and the
samples of a group of pictures are placed in one MOQT Group.

The LOC sample model is insufficient for V3C.  Atlas data does not
have the semantics of an encoded video sample represented by the
WebCodecs `EncodedVideoChunk` interface {{W3C-WEBCODECS}}.  Instead,
it carries reconstruction metadata, including patch placement and
the mapping between 3D geometry and the 2D component videos.
Occupancy, geometry, attribute, packed, basemesh, and displacement
bitstreams, when present, are distinct V3C components.  Carrying
those components on separate tracks allows a receiver to obtain them
with SUBSCRIBE or FETCH and to prioritize or omit them independently.
Such selection remains subject to the component presence and
dependencies signaled by the active VPS and the catalog; a receiver
that omits a required component cannot reconstruct the corresponding
volumetric frame.  The VPS is sequence-level decoder configuration,
not temporal media.  Consistent with that role, {{RFC10034}} signals
the VPS out of band using `sprop-v3c-parameter-set`, rather than
carrying it as a media packet stream.

This document therefore defines a V3C packaging value for MSF,
analogous to the way {{I-D.ietf-moq-cmsf}} adds CMAF packaging on top
of MSF.  It operates at the streaming-format layer and does not
modify MOQT.  {{ISOIEC23090-5}} specifies the
V3C bitstream, VPS, and unit types.  {{ISOIEC23090-10}} specifies
carriage of V3C data in the ISO Base Media File Format and Dynamic
Adaptive Streaming over HTTP (DASH).  {{RFC10034}} specifies carriage
of V3C atlas data over RTP.  This document specifies carriage of the
same V3C components over MOQT and their description in the MSF
catalog.

This document is written against draft-ietf-moq-transport-23 and thus uses
MOQT version 23 terminology, including Delivery Mode,
Track Properties and Object Properties, SUBSCRIBE_TRACKS,
REQUEST_UPDATE, PUBLISH_DONE, and the data-plane distinction between
Subgroup stream delivery and Datagram delivery.  {{I-D.ietf-moq-msf}}
is written against an earlier MOQT version and requires a Joining
FETCH to obtain the catalog; draft-ietf-moq-transport-23 replaces
Joining FETCH with fill fetch streams requested using FILL_PARAMETERS.
This document uses the MOQT version 23 mechanism ({{join}}).

This document does not redefine V3C bitstream syntax or semantics,
and it does not specify a video or mesh codec for V3C components.

When Datagram delivery is used, {{datagram-encoding}} requires the
complete MOQT Object to fit within the datagram size limit of the
underlying QUIC or WebTransport session.  To satisfy this
requirement, an encoder can limit the sizes of slices and NAL units.
{{datagram-options}} compares alternative approaches and lists
questions for Working Group consideration.

# Terminology

{::boilerplate bcp14-tagged}

This document uses the following terms:

V3C component:
: Atlas, occupancy, geometry, or attribute data of a particular type
  associated with a V3C volumetric content representation, as defined
  in {{ISOIEC23090-5}} and restated in {{RFC10034}}.  Packed video
  data, common atlas data, basemesh data, and arithmetic-coded
  displacement data, when present, are also treated as V3C components
  in this mapping.

V3C parameter set (VPS):
: A syntax structure containing syntax elements that apply to zero or
  more coded V3C sequences, as defined in {{ISOIEC23090-5}}.  The VPS is
  configuration, not a temporal media component.  Among other
  decoder resources, it maps each attribute index to an attribute
  type ({{vps-attribute-types}}) and indicates which component
  sub-bitstreams are present ({{component-presence}}).

Attribute:
: A scalar or vector property optionally associated with each point
  in a volumetric frame, such as color, reflectance, surface normal,
  transparency, or material ID ({{ISOIEC23090-5}}).

V3C unit:
: A V3C unit header and payload as defined in {{ISOIEC23090-5}}.  The
  header identifies the unit type and, when present, atlas, map,
  packed-map, attribute, auxiliary-video, and related indices
  ({{RFC10034}}, Section 4.3.1; {{v3c-track-identifiers}}).

Coded V3C unit:
: The coded data of one V3C component for one V3C composition unit,
  typically one coded atlas access unit or one coded video access
  unit.  Network Abstraction Layer (NAL) units, when used by the
  underlying codec, are contained in this payload; they are not the
  MOQT Object boundary.

V3C composition unit:
: The set of all sub-bitstream composition units that share the same
  composition time, as defined in {{ISOIEC23090-5}}.  One composition
  unit reconstructs one coded volumetric frame.

IRAP composition unit:
: A V3C composition unit in which every carried component starts with
  an intra random access point (IRAP), as defined in
  {{ISOIEC23090-5}}.

MOQT Track, Group, Subgroup, and Object:
: The hierarchical data model elements defined by
  {{I-D.ietf-moq-transport-23}} for naming, ordering, and transporting
  media objects.

Delivery Mode:
: An enumeration associated with a MOQT object that indicates whether
  the object is sent on a subgroup stream (Subgroup) or as a single
  object datagram (Datagram).  The Original Publisher establishes an
  object's Delivery Mode by how it first transmits the object.  The
  Delivery Mode is a property of an individual object and can vary
  among objects in the same track.  In a subscription, an object is
  sent according to its Delivery Mode
  ({{I-D.ietf-moq-transport-23}}, Sections 2.1 and 2.1.1).

MSF Catalog:
: The MSF catalog track defined in {{I-D.ietf-moq-msf}}, which
  advertises tracks, relationships, selection properties, and
  initialization data.

# Relationship to MSF

All requirements and terminology in {{I-D.ietf-moq-msf}} apply unless
this document states otherwise.  This specification adds:

* the catalog packaging value `v3c` ({{v3c-packaging-type}});
* rules for carrying the VPS and component decoder configuration in
  the catalog Initialization Data List ({{initialization}});
* recovery of attribute type from that VPS
  ({{vps-attribute-types}});
* a track-level `v3c` object that copies identifiers from the V3C
  unit header ({{v3c-track-identifiers}});
* two Object payload framings: native framing using only
  {{ISOIEC23090-5}} syntax, and ISOBMFF framing using the sample
  formats of {{ISOIEC23090-10}} ({{payload-format}});
* a mapping from V3C components, composition units, and coded units
  onto MSF tracks, MOQT Groups, and MOQT Objects ({{mapping}}).

A V3C component track MUST NOT use the MSF packaging value `loc`.
LOC packaging remains available for ordinary audio and video tracks
in the same catalog, for example an accompanying 2D video or audio
commentary track.

# Architecture
{: #architecture}

V3C syntax stays at the streaming-format layer.  MOQT carries opaque
objects.  Relays forward using MOQT Track, Group, Subgroup, Object,
priority, and timeout metadata only.  {{fig-architecture}} shows the
layering.  {{tbl-mapping}} summarizes the mapping.

~~~
                    V3C content
                         |
          +--------------+---------------+
          |                              |
    V3C configuration               V3C media
          |                              |
         VPS              independently consumable
          |               components present in the VPS
          |                              |
          |                 AD   CAD   OVD   GVD
          |                 AVD  PVD   BMD   ADD
          |
          +------------------+------------------+
                             |
                      MSF packaging
                             |
              +--------------+--------------+
              |              |              |
           Track A        Track B        Track C
              |              |              |
            Groups         Groups         Groups
              |              |              |
            Objects        Objects        Objects
              |              |              |
              +--------------+--------------+
                             |
                            MOQT
                             |
                            QUIC
~~~
{: #fig-architecture title="V3C packaging relative to MSF and MOQT"}

The mapping is:

| V3C | MSF / MOQT |
| :--- | :--- |
| VPS | `initDataList` entry referenced by the atlas track `initRef`: `v3c_parameter_set()` (native framing) or `V3CDecoderConfigurationRecord` (ISOBMFF framing) |
| Component decoder configuration | In band (native framing), or `initDataList` entry referenced by the component track `initRef` (ISOBMFF framing) |
| Each independently consumable component (AD, OVD, GVD, AVD, PVD, CAD, BMD, ADD) | One MSF track |
| Run of V3C composition units starting at an IRAP composition unit | One MOQT Group, with identical Group IDs across time-aligned tracks |
| Coded V3C unit (one component, one composition unit) | One MOQT Object |
| NAL unit | Bytes inside the object payload |
| Component identity (`vuh_*`) | Catalog `v3c` object |
| Decode dependency | Catalog `depends` |
| Same presentation | Catalog `renderGroup` |
| Alternate quality of one component, including V-DMC GVD and ADD displacement | Catalog `altGroup` |
{: #tbl-mapping title="V3C to MSF/MOQT mapping"}

# Mapping V3C to the MOQT Object Model
{: #mapping}

## Encapsulation Modes
{: #encapsulation-modes}

An encapsulation mode determines how V3C components are assigned to
MOQT tracks.  The body of this document specifies multi-track mode.
Single-track mode, in which all V3C components share one track, is
NOT RECOMMENDED and is specified in {{single-track}}.  Independently
of the encapsulation mode, a payload framing determines the byte
format of an Object payload ({{payload-format}}).

Multi-track mode:
: Each independently consumable V3C component is carried on its own
  MOQT track.  This mode enables selective subscription, independent
  publisher and subscriber priority, and independent congestion
  response per component.  Alternate encodings of the same component
  are separate tracks that share an `altGroup`
  ({{alternate-groups}}).

In either encapsulation mode, subscribers MUST have catalog metadata sufficient to
identify required dependencies (for example atlas data required to
interpret geometry or packed video) before decoding.

Either encapsulation mode MAY mix Delivery Mode Subgroup and Datagram within a
track or group, subject to {{objects-subgroups-datagrams}} and
{{datagram-guidelines}}.

> Editor's note: Datagram delivery may not be recommended and may be
> removed from the scope of the next revision of this document.
{: gi="aside"}

## Tracks
{: #tracks}

In multi-track mode, the publisher exposes a separate track for each
V3C component that it offers, and the
active VPS (including any VPS packed-video extension) indicates that the component is
present ({{component-presence}}).  The offered set MAY include:

* atlas data (V3C_AD);
* atlas tile data (role v3c-atlas-tile), when an atlas frame is partitioned into spatial tiles ({{atlas-tiles}});
* occupancy video data (V3C_OVD), when present;
* geometry video data (V3C_GVD), when present;
* one track per attribute (V3C_AVD) that the publisher offers
  independently.  The attribute type (for example "texture" or
  "surface normal") is not encoded in the track name but included in the
  VPS ({{vps-attribute-types}});
* packed video data (V3C_PVD), when present;
* common atlas data (V3C_CAD), when present;
* basemesh data (V3C_BMD), when present;
* arithmetic-coded displacement data (V3C_ADD), when present.

Occupancy video, geometry video, attribute video, packed video,
common atlas, basemesh, and displacement are optional at the V3C
layer.  A presentation MUST NOT be assumed to contain atlas,
occupancy, geometry, and attribute video tracks together.  Packed
video MAY be the only video component.

In Video-based Dynamic Mesh Coding (V-DMC) {{ISOIEC23090-29}},
basemesh data carries the base mesh.  Displacement vectors are
carried either as geometry video data (V3C_GVD) or as
arithmetic-coded displacement data (V3C_ADD).  For a given atlas,
those two forms are mutually exclusive in the VPS.  A geometry track
in that case carries video-coded displacement; it does not replace
the basemesh track.  Selection between the two forms is specified in
{{alternate-groups}}.

The VPS is not advertised as a continuously running media track
({{initialization}}).

MOQT Track Names are opaque byte sequences
({{I-D.ietf-moq-transport-23}}).  This specification defines the
following recommended UTF-8 naming patterns to make catalog entries
readable.  The slash (`/`, byte `0x2f`) is an ordinary Track Name byte;
it does not create MOQT namespace fields or otherwise have protocol
semantics.  The Track Namespace remains separately encoded.  When a
name is rendered using the safe serialization defined in Section 8.8
of {{I-D.ietf-moq-transport-23}}, each slash is encoded as `.2f`; for
example, `v3c/attribute/0/0/0` is rendered as
`v3c.2fattribute.2f0.2f0.2f0`.  The hyphen is also not rendered
verbatim; `v3c/common-atlas` is rendered as
`v3c.2fcommon.2datlas`.

The angle-bracketed identifiers in these patterns are replaced by
their non-negative decimal values:

~~~
v3c/atlas/<atlasId>
v3c/atlas/<atlasId>/tile/<tileId>
v3c/occupancy/<atlasId>
v3c/geometry/<atlasId>
v3c/geometry/<atlasId>/<mapIndex>
v3c/geometry/<atlasId>/<mapIndex>/aux
v3c/attribute/<atlasId>/<attributeIndex>/<attributePartitionIndex>
v3c/attribute/<atlasId>/<attributeIndex>/<attributePartitionIndex>/<mapIndex>
v3c/attribute/<atlasId>/<attributeIndex>/<attributePartitionIndex>/<mapIndex>/aux
v3c/packed/<atlasId>
v3c/packed/<atlasId>/<packedMapIndex>
v3c/basemesh/<atlasId>
v3c/displacement/<atlasId>
v3c/common-atlas
~~~
{: #fig-track-names title="Recommended V3C track name patterns"}

These names are a convention, not the source of component identity.
A receiver MUST use the fields in the catalog `v3c` object
({{v3c-track-identifiers}}) and MUST NOT require a publisher to use
these recommended names.

Occupancy, atlas, basemesh, and displacement unit headers do not
include `vuh_map_index`; those tracks omit the map-index path
element.  Common atlas data has no `vuh_atlas_id`.  Packed-video
unit headers carry `vuh_packed_map_index` rather than `vuh_map_index`
({{v3c-track-identifiers}}).  Geometry and attribute map splitting is
specified in {{map-streams}}.

Subscribers obtain V3C component tracks using MOQT subscription
procedures.  A subscriber MAY send SUBSCRIBE for an individual
component track, or SUBSCRIBE_TRACKS to request publication of tracks
in a namespace that match advertised catalog metadata and filters.
A subscriber MAY later send REQUEST_UPDATE to pause or resume
forwarding, or to change filters or subscriber priority
({{I-D.ietf-moq-transport-23}}, Section 9.5).  A publisher or relay
terminates an established subscription by sending PUBLISH_DONE.

### Component Presence
{: #component-presence}

A publisher MUST advertise a V3C component track only when the
corresponding sub-bitstream is present according to the active VPS
and, for packed video, the VPS packed-video extension
(`vps_packed_video_present_flag`).  Presence is recovered from the
VPS the same way attribute type is recovered
({{vps-attribute-types}}).  This document does not duplicate VPS
presence flags or profile, tier, and level (PTL) fields such as
`ptl_max_decodes_idc` in the catalog.

Per atlas, {{ISOIEC23090-5}} signals:

* `vps_occupancy_video_present_flag`
* `vps_geometry_video_present_flag`
* `vps_attribute_video_present_flag`
* `vps_auxiliary_video_present_flag`
* `vps_packed_video_present_flag` (in a VPS packed-video extension)

When packed video is the only video component, occupancy, geometry,
and attribute video tracks are omitted.  Packed tracks still list
the corresponding atlas track in `depends` ({{dependencies}}).

### Map Streams
{: #map-streams}

`vps_multiple_map_streams_present_flag` in the VPS controls whether
maps of a geometry or attribute component occupy one video stream or
several ({{ISOIEC23090-5}}):

* When the flag is 0, all maps of that component for an atlas are in
  one video stream.  The publisher advertises one MOQT track for that
  component and omits `v3c.mapIndex` (the map index is derived as in
  {{ISOIEC23090-5}}).
* When the flag is 1, each map is its own stream.  The publisher
  advertises one MOQT track per `mapIndex`.

Multiple maps are a generic VPS feature.  Extra maps are used heavily
by some V3C applications; the track rule above is not application-
specific.

### Auxiliary Sub-Bitstreams
{: #auxiliary}

When `vps_auxiliary_video_present_flag` is 1, a geometry or attribute
component MAY include a distinct auxiliary video sub-bitstream,
identified by `vuh_auxiliary_video_flag` equal to 1 in the V3C unit
header.  That sub-bitstream is a separate independently consumable
component: the publisher advertises a separate MOQT track and sets
`v3c.auxiliaryVideo` to 1 ({{v3c-track-identifiers}}).  Two geometry
(or attribute) tracks that share atlas and map identifiers are
distinguished by this field.  A separate auxiliary track MUST NOT be
advertised when the flag is 0.

### Atlas Tiles
{: #atlas-tiles}

{{ISOIEC23090-5}}, Clause 7.4, permits an atlas frame to be
partitioned into non-overlapping rectangular tiles.  One atlas coding
layer (ACL) NAL unit carries one tile.  Tiles MAY align with video
slices or subpictures.  {{RFC10034}} signals a tile subset with
`sprop-v3c-tile-id`.  {{ISOIEC23090-10}} defines optional atlas tile
tracks.

The default mapping in this document is that all tiles of an atlas
stay in one atlas track.  One coded atlas access unit is one MOQT
Object.

A publisher MAY expose atlas tile tracks.  An atlas tile track
carries only the ACL NAL units of its tile; the atlas parameter sets
and the VPS remain in the atlas track ({{ISOIEC23090-10}},
Clause 7.3.2.2).  An atlas tile track includes `v3c.tileId`, lists
the atlas track in `depends`, and references the same `initDataList`
entry as the atlas track.  Video, basemesh, or displacement tracks
that are spatially aligned with that tile SHOULD also list the tile
track in `depends`.  Per-tile tracks are not required.  This document
does not require viewport-driven delivery.

## Initialization Data
{: #initialization}

The VPS MUST be provided as MSF initialization data, not as a
continuously running media track.  This matches the out-of-band VPS
treatment in {{RFC10034}} (`sprop-v3c-parameter-set`).  Each
initialization data item is carried in the catalog `initDataList`
({{I-D.ietf-moq-msf}}, Section 5.1.7) as an entry with `type` equal
to `inline`, whose `data` value is the Base64 encoding {{RFC4648}} of
the item.  The items depend on the payload framing of the track
({{payload-format}}).

With native framing, the atlas track references with `initRef`
({{I-D.ietf-moq-msf}}, Section 5.2.13) an entry that contains the `v3c_parameter_set()`
syntax structure of the active VPS, without a V3C unit header.  This
is the same value as `sprop-v3c-parameter-set` in {{RFC10034}}.
Atlas parameter sets, SEI messages, and the parameter sets of the
video, basemesh, and displacement sub-bitstreams are carried in band
in the Objects ({{vps}}).  Video, basemesh, and displacement tracks
that use native framing do not have an `initRef`.

With ISOBMFF framing, each V3C-packaged track references one decoder
configuration record with `initRef`.  The record is the structure
that {{ISOIEC23090-10}} or {{ISOIEC14496-15}} places in the
corresponding ISOBMFF sample entry, without the enclosing box header:

| Track | Decoder configuration record |
| :--- | :--- |
| Atlas (V3C_AD), common atlas (V3C_CAD) | `V3CDecoderConfigurationRecord` with version 1 ({{ISOIEC23090-10}}, Clause 6.2.1) |
| Atlas tile | The record of the atlas track that the tile track depends on |
| Video (V3C_OVD, V3C_GVD, V3C_AVD, V3C_PVD) | Decoder configuration record of the video codec, for example `HEVCDecoderConfigurationRecord` ({{ISOIEC14496-15}}) |
| Basemesh (V3C_BMD) | `BaseMeshDecoderConfigurationRecord` ({{ISOIEC23090-10}}, Clause 6.2.2) |
| Displacement (V3C_ADD) | `ACDisplacementDecoderConfigurationRecord` ({{ISOIEC23090-10}}, Clause 6.2.3) |
{: #tbl-init title="Decoder configuration record referenced by initRef"}

The `V3CDecoderConfigurationRecord` of the atlas track carries the
active VPS as a complete V3C unit of type V3C_VPS, including its V3C
unit header ({{ISOIEC23090-10}}, Clause 6.2.1).  This differs from
`sprop-v3c-parameter-set` in {{RFC10034}}, which carries only the
`v3c_parameter_set()` syntax structure.  The record MAY also carry
atlas parameter sets and SEI messages in its setup unit arrays.

Other V3C component tracks do not reference the VPS directly.  Each
such track lists the atlas track in `depends` ({{dependencies}}), and
a receiver obtains the VPS from the initialization data of that atlas
track.  The
`id` of each `initDataList` entry MUST be unique within the catalog.
Tracks that use identical decoder configuration MAY reference the
same entry.

### Contents of the VPS
{: #vps}

This subsection restates the role of the VPS for the MOQT mapping.
It does not redefine VPS syntax.  Syntax and semantics remain as
specified in {{ISOIEC23090-5}} and as summarized in {{RFC10034}},
Section 4.2.

The VPS is carried in a V3C unit of type V3C_VPS.  It applies to
zero or more coded V3C sequences.  Media-component unit headers
identify the active VPS with `vuh_v3c_parameter_set_id`, which
equals `vps_v3c_parameter_set_id` of that VPS ({{RFC10034}},
Section 4.3.1).  Catalog `v3c.parameterSetId`, when present, copies
the same identifier.

One VPS governs the components of the coded V3C sequence (CVS) it
applies to.  Atlas data (V3C_AD), geometry video data (V3C_GVD),
attribute video data (V3C_AVD), occupancy video data (V3C_OVD), and
packed video data (V3C_PVD) do not use a separate parameter set for
each media type.  Every such unit carries `vuh_v3c_parameter_set_id`
and thereby refers to that same active VPS.  The catalog records
that VPS once, in the initialization data of the atlas track
({{initialization}}).

The VPS is sequence-level configuration for the CVS, not a
per-component parameter set.  It includes the atlas geometry configuration in
`geometry_information()` (codec identifiers and the 2D and 3D bit
depths), and attribute configuration in `attribute_information()`.
`attribute_information()` maps each attribute index to an attribute
type, such as texture or, for a Gaussian splatting representation,
spherical harmonics, scale, rotation, or opacity
({{vps-attribute-types}}). This document does not specify Gaussian
reconstruction.

A receiver uses the VPS to determine the resources required to
decode and reconstruct the bitstream: which atlas, occupancy,
geometry, attribute, packed, auxiliary, basemesh, and displacement
components are present, how they relate, and which attribute type
each AVD component carries ({{vps-attribute-types}}).  Placing the
VPS in catalog initialization data is the MOQT counterpart of
carrying it out of band in SDP as `sprop-v3c-parameter-set`
({{RFC10034}}).

VPS extensions (packed video, MIV, V-DMC) travel inside the VPS
bytes.  This document does not parse those extensions in the
catalog.  Profile, tier, and level, including `ptl_max_decodes_idc`,
are recovered from the VPS; they are not duplicated as catalog
fields.

With native framing, atlas sequence parameter sets (ASPS), atlas
frame parameter sets (AFPS), and Supplemental Enhancement Information
(SEI) messages are carried in the atlas Objects.  Because every Group
starts with an IRAP composition unit ({{groups}}), the first Object
of each Group of a track that uses native framing MUST contain the
parameter sets needed to decode that track from that Object.

With ISOBMFF framing, ASPS, AFPS, and SEI messages MAY be carried in
the setup unit arrays of the `V3CDecoderConfigurationRecord`, in the
atlas Objects, or in both, subject to the `array_completeness` values
of the record ({{ISOIEC23090-10}}, Clause 6.2.1).  A publisher that
does not mark those arrays complete SHOULD repeat the parameter sets
in the first atlas Object of each Group.

Video-codec configuration for OVD, GVD, AVD, or PVD (for example HEVC
or VVC parameter sets) is not part of the VPS.  With native framing,
it is carried in band in the video sub-bitstream.  With ISOBMFF
framing, it is carried in the decoder configuration record that the
video track references with `initRef`.  Basemesh and displacement
configuration is likewise carried in band with native framing, or in
the records listed in {{tbl-init}} with ISOBMFF framing; payload
syntax for those components is specified in {{ISOIEC23090-29}}.

### Attribute Types in the VPS
{: #vps-attribute-types}

An attribute is a property of reconstructed volumetric primitives.
Examples include texture (color), material ID, transparency,
reflectance, surface normal, spherical harmonics, scale, and rotation
({{ISOIEC23090-5}}, Table 5).

The catalog identifies an AVD track by `v3c.attributeIndex`, which
copies `vuh_attribute_index` from the V3C unit header.  That index
is not itself the attribute type.  The VPS maps each
`vuh_attribute_index` to the type of attribute carried by the
corresponding AVD component.  A V3C decoder uses that mapping to
identify which attribute a given video component contains, for
example color ({{RFC10034}}, Section 4.3.1).

~~~
VPS (atlas track initRef)
    |
    |  attribute_index 0 -> texture (color)
    |  attribute_index 1 -> surface normal
    v
AVD tracks
    v3c/attribute/0/0/0    v3c.attributeIndex = 0
    v3c/attribute/0/1/0    v3c.attributeIndex = 1
~~~
{: #fig-vps-attr title="Attribute type is recovered from the VPS"}

In the recommended track name
`v3c/attribute/<atlasId>/<attributeIndex>/<attributePartitionIndex>`,
the three numbers identify:

* `atlasId`: the atlas to which the attribute belongs;
* `attributeIndex`: the zero-based attribute entry in that atlas's VPS
  attribute information; and
* `attributePartitionIndex`: the zero-based partition of that
  attribute.

Thus, `v3c/attribute/0/0/0` means atlas 0, attribute 0, partition 0.
The VPS says what attribute 0 represents, such as texture (color).
Likewise, `v3c/attribute/0/1/0` means atlas 0, attribute 1,
partition 0.  If attribute 0 were split into two partitions, their
recommended names would end in `/0/0` and `/0/1`.  These names are a
readable convention; receivers use the corresponding fields in the
catalog `v3c` object rather than parsing the track name.

A subscriber that needs a particular attribute type (for example
color, and not surface normals) parses the referenced VPS and
subscribes to the AVD track or tracks whose `attributeIndex` maps
to that type.  This document does not duplicate attribute-type
names in the catalog.  The VPS is authoritative.

An attribute MAY be partitioned across several AVD components
when, for example, the attribute has more dimensions than the
video codec can code in one stream ({{RFC10034}}, Section 7.3).
The VPS describes that partitioning.  Catalog
`v3c.attributePartitionIndex` identifies which partition a track
carries.

### Join Sequence
{: #join}

Joining a V3C presentation uses the catalog, then initialization
data, then media, as shown in {{fig-join}}.  Fill fetch streams are
requested with FILL_PARAMETERS ({{I-D.ietf-moq-transport-23}},
Section 3.4):

~~~
Subscriber
    |
    | SUBSCRIBE catalog (FILL_PARAMETERS to Largest Object)
    v
Catalog
    |
    | atlas initRef -> VPS
    | other initRef -> component decoder configuration
    |                  (ISOBMFF framing only)
    v
V3C initialization
    |
    | SUBSCRIBE component tracks at the latest Group
    | (Location Filter + FILL_PARAMETERS)
    v
Time-aligned Groups
    |
    v
Decode
~~~
{: #fig-join title="Join sequence for a V3C presentation"}

A subscriber MUST NOT begin decoding V3C component tracks until it
has obtained the VPS and, for tracks that use ISOBMFF framing, the
decoder configuration record of each subscribed track.

In multi-track mode, the VPS is carried only in initialization data;
{{single-track}} specifies an exception for single-track mode.  If
the active VPS changes, the publisher MUST
publish a catalog update with a new `initDataList` entry and updated
`initRef` values.

## Groups and Random Access
{: #groups}

One MOQT Group carries a run of one or more consecutive V3C
composition units that begins with an IRAP composition unit.  Each
Group is therefore a join point at which a subscriber can start
decoding, consistent with the MOQT requirement that Groups be
independently useful ({{I-D.ietf-moq-transport-23}}, Section 2.3)
and with the MSF requirement that a group of pictures be placed
within one MOQT Group ({{I-D.ietf-moq-msf}}, Section 4.1).  Within a
Group, each Object carries the coded V3C unit of one composition
unit, in decoding order ({{objects-subgroups-datagrams}}).

Component tracks that belong to the same `renderGroup` and that
reconstruct the same volumetric frames MUST use identical MOQT Group
IDs and identical Object IDs for corresponding composition units.
This requirement is stricter than the time-alignment guidance in
{{I-D.ietf-moq-msf}}, Section 4.2, and allows a receiver to associate
the coded V3C units of one composition unit without parsing the
payloads.

Every Group MUST begin with an IRAP composition unit: the first
Object of every carried component is a random access point.  For
atlas and common-atlas data, that is an IRAP coded atlas access unit
(an atlas access unit whose ACL NAL units have a NAL_IDR_W_RADL, or NAL_IDR_N_LP, NAL_GIDR_W_RADL, NAL_GBLA_N_LP, NAL_GCRA or NAL_GIDR_N_LP 
NAL unit type as specified in {{ISOIEC23090-5}}).  For video-coded
components, it is the IRAP defined by the video codec in use, for
example an Instantaneous Decoding Refresh (IDR) or Clean Random
Access (CRA) picture.  For a non-video component, it is the random
access point defined by that component's coding specification.  A
coded V3C sequence (CVS) starts with a VPS, which can be provided out
of band, and its first composition unit is an IRAP composition unit
({{ISOIEC23090-5}}).

A coded V3C unit MUST NOT span Groups.

Example of cross-track alignment, with two Groups of two composition
units each:

~~~
Group 100   (starts with an IRAP composition unit)
    v3c/atlas/0              Object 0: AD[0]    Object 1: AD[1]
    v3c/occupancy/0          Object 0: OVD[0]   Object 1: OVD[1]
    v3c/geometry/0           Object 0: GVD[0]   Object 1: GVD[1]
    v3c/attribute/0/0/0      Object 0: AVD[0]   Object 1: AVD[1]

Group 101   (starts with an IRAP composition unit)
    v3c/atlas/0              Object 0: AD[2]    Object 1: AD[3]
    v3c/occupancy/0          Object 0: OVD[2]   Object 1: OVD[3]
    v3c/geometry/0           Object 0: GVD[2]   Object 1: GVD[3]
    v3c/attribute/0/0/0      Object 0: AVD[2]   Object 1: AVD[3]
~~~
{: #fig-group-align title="Equal Group and Object IDs align V3C components in time"}

## Objects, Subgroups, and Datagrams
{: #objects-subgroups-datagrams}

A MOQT Object carries one coded V3C unit: the coded data of the
track's component for one V3C composition unit.  Object IDs within a
Group follow the decoding order of the composition units.  For a
video-coded component, the Object is one coded video access unit.
For atlas or common-atlas data, it is one coded atlas access unit,
which contains zero or more atlas NAL units.  For basemesh or
displacement data, it is one coded access unit of that component.

NAL units are an underlying codec structure.  They remain inside the
object payload.  Publishers SHOULD NOT place each NAL unit in its
own MOQT Object solely to express codec framing.  When datagram
delivery is used, slice and NAL unit size is constrained at encode
time ({{datagram-encoding}}).

In multi-track mode, the four-byte V3C unit header is not carried in
the object.  Header fields needed to identify the component are
advertised in the
catalog `v3c` object ({{v3c-track-identifiers}}).  The payload format
is specified in {{payload-format}}.

MOQT defines a Delivery Mode for each object
({{I-D.ietf-moq-transport-23}}, Section 2.1.1).  The Delivery Mode can
be Subgroup or Datagram, and it can vary among objects in the same
track.  The Original Publisher establishes the Delivery Mode by how it
first transmits the object, and in a subscription an object MUST be
sent according to its Delivery Mode.

When an object has Delivery Mode Subgroup, it is
delivered on a MOQT subgroup stream.  Subgroups provide ordered
delivery on a single QUIC stream.  They MAY distinguish independently
discardable layers of the same component, for example a base layer
and an enhancement layer of geometry:

~~~
Track: v3c/geometry/0/0
Group 100
    Subgroup 0 (base)
        Object 0
        Object 1
    Subgroup 1 (enhancement)
        Object 0
        Object 1
~~~
{: #fig-subgroups title="Subgroups as layers of one V3C component"}

When no such layering is offered, a publisher MAY place every object
of the Group on subgroup 0.

In multi-track mode, subgroups MUST NOT be used as the primary means
of separating V3C component types; those are separate tracks
({{tracks}}).

When an object has Delivery Mode Datagram, it is
conveyed as a single MOQT object datagram.  Such an object is not sent
in a subgroup and has no Subgroup ID.  Datagram delivery is suitable
for V3C coded units where low latency is preferred over reliable
delivery, especially when the unit is independently useful,
replaceable by newer data, or can be skipped without preventing
future decoding or rendering.

A sender using datagram delivery MUST ensure that the complete MOQT
object header, any object properties, and the object payload fit
within the maximum datagram size for the session.  If the total object
size exceeds the maximum datagram size, the object will be dropped
without explicit notification ({{I-D.ietf-moq-transport-23}},
Section 11.2).  That session maximum is bounded by the underlying
QUIC or WebTransport datagram limit ({{datagram-encoding}}).
Application-level fragmentation of one coded V3C unit across multiple
MOQT datagrams is unspecified.  For V3C units that may exceed the
datagram size, or for data that requires reliable in-order delivery,
the publisher MUST use subgroup stream delivery instead of datagram
delivery.

The RTP payload format for V3C {{RFC10034}} defines packetization of
one or more V3C atlas NAL units and fragmentation of a V3C atlas NAL
unit across RTP packets.  This document does not define RTP
packetization inside MOQT objects and does not define a V3C
datagram fragment header.

## Object Payload Format
{: #payload-format}

{{ISOIEC23090-5}} specifies the V3C unit stream and the NAL unit
stream, and leaves the framing of those units to the application
({{ISOIEC23090-5}}, Clauses 7.2 and 7.3).  This document defines two
payload framings:

Native framing:
: The Object payload consists only of syntax structures specified in
  {{ISOIEC23090-5}} and {{ISOIEC23090-29}} ({{native-framing}}).  No
  ISOBMFF structure is used.

ISOBMFF framing:
: The Object payload is identical to the corresponding ISOBMFF sample
  specified in {{ISOIEC23090-10}} ({{isobmff-framing}}).  This allows
  conversion between MOQT Objects and ISOBMFF samples without
  rewriting payload bytes.

The framing of a track is signaled by `v3c.framing`
({{v3c-track-identifiers}}).  Native framing is the default.  All V3C
tracks in the same `renderGroup` MUST use the same framing.

> Editor's note: The name, values, and default of `v3c.framing` are
> proposals for discussion.
{: gi="aside"}

### Native Framing
{: #native-framing}

In multi-track mode, an Object payload is the V3C unit payload of one
V3C unit, without its four-byte V3C unit header ({{ISOIEC23090-5}},
Clause 8.3.2.3):

| Component | Object payload |
| :--- | :--- |
| AD, CAD | `atlas_sub_bitstream()` ({{ISOIEC23090-5}}, Clause 8.3.2.4): a `sample_stream_nal_header()` followed by `sample_stream_nal_unit()` structures ({{ISOIEC23090-5}}, Annex D) |
| AD (atlas tile track) | As for the atlas track, containing only the ACL NAL units of the carried tiles ({{atlas-tiles}}) |
| OVD, GVD, AVD, PVD | `video_sub_bitstream()`: a video unit stream in which unit boundaries are identifiable from patterns in the data, in the format identified by `PtlVideoCodecGroupIdc` or by a component codec mapping SEI message ({{ISOIEC23090-5}}, Clause 8.4.2.3) |
| BMD | `basemesh_sub_bitstream()` in the NAL sample stream format identified by 4CC `bmsf` ({{ISOIEC23090-29}}) |
| ADD | `displacement_sub_bitstream()` in the NAL sample stream format identified by 4CC `dacf` ({{ISOIEC23090-29}}) |
{: #tbl-payload-native title="Object payload with native framing"}

An atlas, common-atlas, or atlas tile Object payload begins with its
own `sample_stream_nal_header()`, so the size of each
`ssnu_nal_unit_size` field is known without initialization data.  A
video Object payload uses the framing of the video unit stream, for
example the byte stream format of the video codec with start codes.

<!-- TODO: confirm with ISO/IEC 23090-29 whether each basemesh and displacement V3C unit payload begins with its own sample_stream_nal_header(). -->

### ISOBMFF Framing
{: #isobmff-framing}

With ISOBMFF framing, an Object payload in multi-track mode is
identical to the corresponding ISOBMFF sample of {{ISOIEC23090-10}}.
The size of the length field is taken from the decoder configuration
record that the track references with `initRef`
({{initialization}}).

| Component | Object payload | Length field size from |
| :--- | :--- | :--- |
| AD, CAD | `V3CAtlasSample` ({{ISOIEC23090-10}}, Clause 7.3.2.3) | `unit_size_precision_bytes_minus1` of `V3CDecoderConfigurationRecord` |
| AD (atlas tile track) | `V3CAtlasSample`, containing only the ACL NAL units of the carried tiles | as for the atlas track ({{atlas-tiles}}) |
| OVD, GVD, AVD, PVD | Length-prefixed video sample as specified for the video codec in {{ISOIEC14496-15}} | Decoder configuration record of the video codec, for example `lengthSizeMinusOne` of `HEVCDecoderConfigurationRecord` |
| BMD | `BaseMeshSample` ({{ISOIEC23090-10}}, Clause 7.6.3) | `unit_size_precision_bytes_minus1` of `BaseMeshDecoderConfigurationRecord` |
| ADD | `ACDisplacementSample` ({{ISOIEC23090-10}}, Clause 7.6.3) | `unit_size_precision_bytes_minus1` of `ACDisplacementDecoderConfigurationRecord` |
{: #tbl-payload title="Object payload with ISOBMFF framing"}

An atlas Object MAY have a zero-length payload; as in
{{ISOIEC23090-10}}, such an Object documents the timing of a
composition unit for which no atlas NAL unit is present.  Video,
basemesh, and displacement payloads do not use start codes.

## Datagram Delivery Guidelines for V3C
{: #datagram-guidelines}

MOQT datagrams provide low-latency unreliable delivery of individual
objects.  A V3C publisher SHOULD consider datagram delivery for:

* coded V3C units that fit within the negotiated datagram size and
  that remain useful under loss;
* viewport-driven tile, level-of-detail, or enhancement data that can
  be skipped under loss;
* late or time-sensitive updates where retransmission would no longer
  be useful;
* optional attribute or auxiliary data where omission degrades quality
  but does not prevent continued rendering.

A V3C publisher SHOULD prefer subgroup stream delivery for:

* random-access information required at group boundaries;
* coded video access units or other payloads likely to exceed the
  datagram size;
* any object for which reliable delivery and ordered processing are
  required by the application.

For datagram-delivered V3C objects, publishers SHOULD set delivery
timeouts consistent with the media timeline.  In MOQT,
`OBJECT_DELIVERY_TIMEOUT` causes expired datagrams to be dropped.  For
objects with Delivery Mode Datagram,
`SUBGROUP_DELIVERY_TIMEOUT` acts the same way as
`OBJECT_DELIVERY_TIMEOUT`; if both are non-zero, the smaller value is
used ({{I-D.ietf-moq-transport-23}}, Section 5.2).  This behavior allows
obsolete V3C objects to be discarded instead of increasing latency.

Datagram objects remain schedulable MOQT objects and therefore
participate in MOQT priority scheduling.  A V3C publisher SHOULD
assign numerically lower publisher priority values to
reconstruction-critical objects, such as atlas and base geometry or
basemesh, and
numerically higher priority values to enhancement objects, such as
high-resolution attributes.  Subscribers MAY use subscriber priority
and filters to adjust delivery based on viewport, bandwidth, or
rendering needs, including via REQUEST_UPDATE.

### Encoding Considerations for Datagram Delivery
{: #datagram-encoding}

QUIC DATAGRAM frames cannot be fragmented {{RFC9221}}.  When a
publisher intends to send a coded V3C unit with Delivery Mode
Datagram, the encoder and the MOQT packager therefore share
a size budget: the OBJECT_DATAGRAM that carries the unit, including
its object header, object properties, and payload, MUST fit in a
single datagram.

The applicable budget is the maximum datagram size of the sending
MOQT session, which is itself limited by the underlying transport:

* Native QUIC: an endpoint MUST NOT send a DATAGRAM frame larger than
  the `max_datagram_frame_size` transport parameter advertised by its
  peer ({{RFC9221}}, Section 3).  That parameter is the maximum size
  of a DATAGRAM frame, including the frame type, length, and payload.
  {{RFC9221}} further reduces the usable payload by
  `max_udp_payload_size` {{RFC9000}} and by the path Maximum
  Transmission Unit (MTU), because a DATAGRAM frame that does not fit
  in one QUIC packet cannot be sent.

* WebTransport: the application is provided with the maximum datagram
  size it can send ({{I-D.ietf-webtrans-overview}}, Section 4.2).
  WebTransport over HTTP/3 requires a non-zero
  `max_datagram_frame_size` ({{I-D.ietf-webtrans-http3}}, Section 3)
  and places the application payload in an HTTP Datagram
  {{I-D.ietf-webtrans-http3}}.  HTTP Datagram framing consumes part of
  the QUIC DATAGRAM frame, so the application payload is smaller than
  `max_datagram_frame_size`.

Each MOQT session along the path between the original publisher and
the end subscriber can have a different maximum datagram size.
Relays can also add Object Properties, which increase the encoded
object ({{I-D.ietf-moq-transport-23}}, Section 11.2).  A publisher
SHOULD treat the local session maximum as an upper bound and leave
headroom for the OBJECT_DATAGRAM header, object properties, QUIC or
HTTP Datagram framing, and downstream sessions.

When datagram delivery is used, a publisher SHOULD configure the
video and atlas encoders so that:

* each coded slice and each NAL unit is small enough that, after V3C
  packaging and MOQT object framing, it remains within that budget;
* the coded V3C unit placed in one MOQT Object, typically one access
  unit that may contain one or more NAL units, likewise remains
  within that budget.

Encoder controls that reduce slice and NAL unit size include limiting
the coded data per slice or atlas tile.  This document does not
specify numeric encoder parameters.  Constraining slice and NAL unit
size is encoder configuration.  It does not change the object mapping
in {{objects-subgroups-datagrams}}: NAL units remain inside the
object payload, and publishers still SHOULD NOT emit one MOQT Object
per NAL unit solely to express codec framing.

If a slice or NAL unit cannot be encoded within the budget, or if the
resulting coded V3C unit remains larger than the budget, the
publisher MUST use subgroup stream delivery for that object
({{objects-subgroups-datagrams}}).  This document does not define
fragmentation of a NAL unit or coded V3C unit across MOQT datagrams.
{{datagram-options}} lists alternatives to that rule.

# V3C Packaging
{: #packaging}

This document defines the V3C packaging for carrying coded V3C units
in MOQT Objects using MSF.  When a track uses V3C packaging, the
catalog `packaging` attribute ({{I-D.ietf-moq-msf}}, Section 5.2.4)
MUST be present and MUST have the value `v3c`
({{v3c-packaging-type}}).

Object payloads are the coded V3C units specified in
{{payload-format}}.  The payload is not a LOC sample and does not
include a LOC header.  V3C identifiers used for track selection are
signaled in the track-level catalog object `v3c`
({{v3c-track-identifiers}}) and MUST NOT be carried in MOQT Object
Properties.  MOQT Object Properties remain limited to information
relevant to Object distribution ({{I-D.ietf-moq-transport-23}},
Section 11.1.2).

V3C-packaged tracks MAY carry the following properties defined by
{{I-D.ietf-moq-loc}}:

TIMESCALE (0x08):
: The number of TIMESTAMP units per second, for example 1000 or 90000
  ({{I-D.ietf-moq-loc}}, Section 2.3.1.2).  When used, it SHOULD be
  sent as a Track Property and SHOULD equal the catalog `timescale`
  value ({{I-D.ietf-moq-msf}}, Section 5.2.21).

TIMESTAMP (0x10):
: The presentation timestamp of the V3C composition unit whose coded
  V3C unit the Object carries, in TIMESCALE units
  ({{I-D.ietf-moq-loc}}, Section 2.3.1.1).  Corresponding Objects of
  time-aligned tracks carry the same value.

These properties allow relays and subscribers to synchronize V3C
tracks with other media and to manage playout buffers without parsing
atlas or video payloads.

# Catalog Signaling for V3C
{: #catalog}

Publishers advertise available tracks using the MSF catalog
{{I-D.ietf-moq-msf}}.  V3C publishers MUST include sufficient
information for subscribers to (1) discover all required component
tracks, (2) obtain the VPS and, with ISOBMFF framing, the decoder
configuration of each track via `initRef`, (3) determine dependencies
between tracks, and (4) select compatible alternatives.

A subscriber uses catalog metadata to identify the V3C tracks it needs. It can subscribe to those tracks individually using SUBSCRIBE, or use SUBSCRIBE_TRACKS to request tracks within a namespace, optionally restricted by applicable MOQT Track Property filters. The subscriber can subsequently modify subscription parameters using REQUEST_UPDATE.

## V3C packaging type
{: #v3c-packaging-type}

This specification extends the allowed packaging values defined in
{{I-D.ietf-moq-msf}} to include one new entry:

| Name | Value | Reference |
| :--- | :---- | :-------- |
| V3C | v3c | This document |
{: #tbl-v3c-packaging title="V3C packaging type"}

Every Track entry in a catalog carrying V3C-packaged media data MUST
declare a `packaging` type value of `v3c`.  Object payloads of such
tracks are as specified in {{packaging}}.

## V3C Track Identifiers
{: #v3c-track-identifiers}

{{I-D.ietf-moq-msf}} allows a producer to add catalog fields that do
not collide with defined MSF names.  This document defines a
track-level object named `v3c`.

Location: Track object.
Required: Mandatory when `packaging` is `v3c`.
JSON type: Object.

The `v3c` object carries identifiers already present in the V3C unit
header as specified in {{ISOIEC23090-5}}.  {{RFC10034}}, Section
4.3.1, copies an earlier header layout for informative purposes.
Where that copy and {{ISOIEC23090-5}} differ, this document follows
{{ISOIEC23090-5}}: unit types V3C_BMD and V3C_ADD are defined, and
the packed-video header carries `vuh_packed_map_index` (8 bits) and
9 reserved bits rather than 17 reserved bits.  A subscriber uses
these identifiers to select tracks before any media object arrives.

`component` is a string and `unitType` is a number.  In multi-track
mode, at least one of `component` or `unitType` MUST be present.
`unitType` copies the 5-bit `vuh_unit_type` value from the V3C unit
header as specified in {{ISOIEC23090-5}}, Table 3.  `unitType` MUST
NOT be 0 (V3C_VPS) or a reserved value.  When both `component` and
`unitType` are present, they MUST identify the same unit type in
{{tbl-v3c-component}}.  `atlasId`, `mapIndex`,
`packedMapIndex`, `attributeIndex`, `attributePartitionIndex`,
`parameterSetId`, `auxiliaryVideo`, and `tileId` are numbers.
`framing` is a string.
Presence follows the V3C unit header except as noted below.

* `atlasId` is omitted for `common-atlas`.
* `mapIndex` is omitted when the unit header does not include
  `vuh_map_index`, and when all maps of that component occupy one
  stream ({{map-streams}}).
* `packedMapIndex` is used for packed video.  It copies
  `vuh_packed_map_index`.  It MAY be omitted when a single packed
  map is present.
* `attributeIndex` and `attributePartitionIndex` are used for AVD.
* `auxiliaryVideo` is used for geometry and attribute tracks.  It
  copies `vuh_auxiliary_video_flag`.  A value of 0 MAY be omitted;
  receivers infer 0 when the field is absent ({{auxiliary}}).
* `parameterSetId` is optional.
* `tileId` is omitted unless the track is an atlas tile track
  ({{atlas-tiles}}).  It copies the atlas tile identifier
  (`afti_tile_id` in {{ISOIEC23090-5}}) of the tile carried by the
  track.  It is not a V3C unit header field.
  <!-- TODO: ISO/IEC 23090-10 tile tracks and RFC 10034 sprop-v3c-tile-id allow several tiles per track; consider a tileIds array. -->
* `framing` is a string that identifies the payload framing of the
  track ({{payload-format}}).  Allowed values are `native` and
  `isobmff`.  When the field is absent, receivers infer `native`.  It
  is not a V3C unit header field.  A receiver MUST NOT decode a track
  whose `framing` value it does not recognize.

`attributeIndex` identifies which AVD component a track carries.
The attribute type associated with that index is in the VPS
({{vps-attribute-types}}), not in this object.

| Name | V3C syntax |
| :--- | :--- |
| unitType | vuh_unit_type (5-bit value) |
| component | vuh_unit_type |
| atlasId | vuh_atlas_id |
| mapIndex | vuh_map_index |
| packedMapIndex | vuh_packed_map_index |
| attributeIndex | vuh_attribute_index |
| attributePartitionIndex | vuh_attribute_partition_index |
| auxiliaryVideo | vuh_auxiliary_video_flag |
| parameterSetId | vuh_v3c_parameter_set_id |
| tileId | afti_tile_id (not a unit header field) |
| framing | None (payload framing, not a V3C syntax element) |
{: #tbl-v3c-fields title="Fields of the catalog v3c object"}

Allowed `component` values are:

| unitType | Value | V3C unit type | Identifier |
| :--- | :--- | :--- | :--- |
| 1 | atlas | Atlas data | V3C_AD |
| 2 | occupancy | Occupancy video data | V3C_OVD |
| 3 | geometry | Geometry video data | V3C_GVD |
| 4 | attribute | Attribute video data | V3C_AVD |
| 5 | packed | Packed video data | V3C_PVD |
| 6 | common-atlas | Common atlas data | V3C_CAD |
| 7 | basemesh | Basemesh data | V3C_BMD |
| 8 | displacement | Arithmetic-coded displacement data | V3C_ADD |
{: #tbl-v3c-component title="Allowed v3c.component values"}

CAD payload syntax is specified in {{ISOIEC23090-12}}.  Basemesh and
displacement payload syntax is specified in {{ISOIEC23090-29}}.  This
document maps those unit types the same way it maps CAD: the unit
type and catalog identity are generic; application reconstruction
is out of scope.  Reserved `vuh_unit_type` values 9 through 31
({{ISOIEC23090-5}}, Table 3) have no `component` value in this
table.

Unknown `v3c` fields MUST be ignored, consistent with MSF catalog
parsing rules.  Unknown `component` values MUST be ignored.

For tracks with `packaging` equal to `v3c`, the MSF `codec` field
({{I-D.ietf-moq-msf}}, Section 5.2.18) carries a value of the
`codecs` parameter as used for the corresponding ISOBMFF track in
{{ISOIEC23090-10}}, rather than a WebCodecs codec string.  The MSF
requirement to use the WebCodecs codec registry applies only to LOC
packaging.  The `codec` value is never `v3c`.

* Atlas tracks use a V3C sample entry type followed by the profile,
  tier, and level elements specified in {{ISOIEC23090-10}}, Annex C,
  for example `v3c1.L90.4.0.1`.  The sample entry type also indicates
  the `array_completeness` of atlas parameter sets in the
  `V3CDecoderConfigurationRecord`: `v3c1` when all atlas parameter
  sets are in the record, and `v3cg` otherwise.  Atlas tile tracks use
  `v3t1` ({{ISOIEC23090-10}}, Clause 7.3.2.2).
* Video component tracks use the restricted video sample entry
  `resv`, the V3C video component scheme `vvvc`, and the original
  sample entry type of the video codec, for example `resv.vvvc.hvc1`
  ({{ISOIEC23090-10}}, Clause 7.3.3 and Annex D).  The original
  sample entry type identifies the decoder configuration record that
  the track references with `initRef`.
* Basemesh tracks use `bmc1` or `bmcg`, and arithmetic-coded
  displacement tracks use `acd1` or `acdg` ({{ISOIEC23090-10}},
  Clause 7.6.3).
  <!-- TODO: align basemesh and displacement codec values with a codecs parameter definition if ISO/IEC 23090-10 Annex C adds one. -->

With native framing, no decoder configuration record is carried.  The
`codec` value then uses the sample entry type that allows parameter
sets outside a decoder configuration record: `v3cg` for atlas
tracks, a video sample entry type that allows in-band parameter sets
(for example `resv.vvvc.hev1` for HEVC {{ISOIEC14496-15}}), `bmcg`,
or `acdg`.  With native framing, the `codec` value
identifies the codec, profile, tier, and level only; it does not
indicate ISOBMFF encapsulation.

> Editor's note: A codec identifier for native framing that does not
> reuse ISOBMFF sample entry types may be preferable.
{: gi="aside"}

## Track Roles
{: #track-roles}

Each V3C component track SHOULD indicate a role that reflects the
carried component.  {{I-D.ietf-moq-msf}} defines reserved roles such
as `video` and `audio`, and permits custom roles that do not collide
with those names.  This document defines the following role strings:

`v3c-atlas`, `v3c-atlas-tile`, `v3c-occupancy`, `v3c-geometry`, `v3c-attribute`,
`v3c-packed`, `v3c-common-atlas`, `v3c-basemesh`, and
`v3c-displacement`.

These roles identify V3C component and unit types, not the semantic
type of an attribute.  {{ISOIEC23090-5}} defines rotation as
`ATTR_ROTATION`, signaled by `ai_attribute_type_id` in the VPS, but
the corresponding media unit remains an Attribute Video Data
(`V3C_AVD`) unit.  Consequently, a separate rotation track uses role
`v3c-attribute` and identifies its attribute with
`v3c.attributeIndex`; this specification does not define a
`v3c-rotation` role.  The same rule applies to texture,
transparency, spherical harmonics, scale, and other V3C attribute
types.  When those attributes are carried as regions of a Packed
Video Data (`V3C_PVD`) unit, the track instead uses role
`v3c-packed`.

## Dependencies and Render Groups
{: #dependencies}

Tracks that are designed to be rendered together as one V3C
presentation SHOULD share the same `renderGroup` value
({{I-D.ietf-moq-msf}}, Section 5.2.11).  `renderGroup` expresses
association among all carried components of the same presentation,
including atlas, occupancy, geometry, attribute, packed, common
atlas, basemesh, and displacement tracks.

`vps_atlas_count_minus1` in the VPS MAY be greater than 0.  CAD has
no `atlasId` and applies to all atlases in the CVS.  When a CAD
track is present, each atlas track SHOULD list that CAD track in
`depends`.  Video, packed, basemesh, and displacement tracks list
their atlas track, not the CAD track, in `depends`.

Tracks that cannot be decoded or reconstructed without another track
MUST list that track in `depends` ({{I-D.ietf-moq-msf}},
Section 5.2.14).  In particular, every video-coded V3C component
track, packed video track, basemesh track, displacement track, and
atlas tile track MUST list the corresponding atlas track.
The VPS is reached through the atlas track: the atlas track
references it with `initRef`, and other component tracks list the
atlas track in `depends`.

A receiver that subscribes to geometry, packed video, or basemesh
without the dependent atlas track, or without the referenced VPS,
cannot reconstruct volumetric frames.  Publishers MAY still offer
such a subscription so that a receiver can cache or preview a
component; reconstruction requirements are unchanged.

## Alternate Groups
{: #alternate-groups}

When multiple tracks provide alternate encodings of the same V3C
component (for example geometry at several bitrates), they MUST share
the same `altGroup` value ({{I-D.ietf-moq-msf}}, Section 5.2.12) and
MUST be time-aligned as required by MSF for alternate groups.
A subscriber SHOULD select at most one track per `altGroup`.

Geometry alternatives and attribute alternatives MUST NOT be forced
into a single `altGroup` solely because they belong to the same V3C
presentation.  Independent `altGroup` values allow a subscriber to
combine, for example, high-bitrate geometry with low-bitrate
attributes.  When selecting alternatives, the subscriber MUST also
select a compatible atlas track and VPS.

{{ISOIEC23090-29}} encodes V-DMC displacement either as geometry
video or as arithmetic-coded displacement.  Those forms are mutually
exclusive for one atlas in the VPS.  When a publisher offers both
forms, the geometry track (V3C_GVD, role `v3c-geometry`) and the
arithmetic-coded displacement track (V3C_ADD, role
`v3c-displacement`) MUST share one `altGroup` and MUST be
time-aligned.  Each track MUST list, in `depends`, the atlas track
and basemesh track that match that displacement form, and that atlas
track MUST reference with `initRef` initialization data whose VPS
indicates that displacement form.  A subscriber MUST select at most one of the two
tracks and MUST select those dependent tracks and that
initialization data.  {{catalog-vdmc}} shows the arithmetic-coded
displacement form.

## Catalog Examples
{: #catalog-examples}

The following examples are informative.  Each describes a 50-frame
presentation at 25 frames per second.  The active VPS determines
which component tracks are present.  The examples use the
recommended naming patterns in {{fig-track-names}}, but MOQT treats
track names as opaque identifiers and does not infer component
semantics from them.  Within one example, every media track shares
`renderGroup` 1, `isLive` false, the same namespace, and the same
media timeline.

Except for {{catalog-native}}, the examples use ISOBMFF framing
({{isobmff-framing}}), and every `v3c` object therefore contains
`"framing": "isobmff"`.  The atlas track references the
`V3CDecoderConfigurationRecord` entry `atlas-init`, and the video
component tracks reference the
`HEVCDecoderConfigurationRecord` entry `hevc-init` ({{initialization}}).
For brevity, all video component tracks in an example share one
video decoder configuration; a publisher whose components use
different video configurations provides one entry per configuration.
The `data` values are placeholders, not working configuration
records.  Attribute types are recovered from the VPS
({{vps-attribute-types}}), not from the track name.

The atlas track `codec` value `v3c1` indicates that all atlas
parameter sets are in the `V3CDecoderConfigurationRecord`; the
profile, tier, and level elements of the codecs value
({{ISOIEC23090-10}}, Annex C) are omitted for brevity.  Video
component tracks use `codec` `"resv.vvvc.hvc1"`:

| Element | Meaning |
| --- | --- |
| `resv` | Restricted video sample entry ({{ISOIEC14496-12}}).  The samples are not plain 2D video. |
| `vvvc` | Restriction scheme for a V3C video component ({{ISOIEC23090-10}}): occupancy, geometry, attribute, or packed video. |
| `hvc1` | The video is HEVC/H.265 with parameter sets carried in the `HEVCDecoderConfigurationRecord` ({{ISOIEC14496-15}}), here referenced by `initRef`. |
{: #tbl-example-codec title="Elements of the example video codec value"}

MSF expresses both `trackDuration` and the template's
`deltaMediaTime` in milliseconds ({{I-D.ietf-moq-msf}}, Section
7.4.1).  For these examples:

~~~
frame duration = 1000 / 25 = 40 ms
frame count = trackDuration / deltaMediaTime
            = 2000 / 40
            = 50
~~~

The template `[0, 40, [1, 0], [0, 1], 0, 0]` therefore describes
entry indices `n` from 0 through 49:

~~~
mediaTime[n] = n * 40 ms
location[n] = [1, n]
~~~

In these examples, the whole presentation is one run of 50
composition units that starts with an IRAP composition unit, so it is
carried in a single Group 1 ({{groups}}).  The first frame starts at
0 ms at Group 1, Object 0.  The last frame starts at 1960 ms at
Group 1, Object 49 and ends at the 2000 ms track duration.  A FETCH
covering all 50 Objects uses a start Location `[1, 0]` and an
inclusive end Location `[1, 49]` ({{I-D.ietf-moq-transport-23}}).
The `timescale` value does not scale these template values; the
template interval is explicitly in milliseconds.

The examples in {{catalog-pack-attr}} and {{catalog-pack-none}}
illustrate Gaussian splatting content conforming to the V-PCC GS or
V-PCC GS Still toolset profile component of {{ISOIEC23090-5}}.
Annex A of {{ISOIEC23090-5}} identifies those profile components and
their profile, tier, and level signaling; Annex H specifies their
V-PCC restrictions and reconstruction processes.  The mapping in
this document is not limited to those profiles.  The packed and
unpacked component organizations are determined by the
component-presence flags and packed-video extension in the active
VPS, not by the catalog.

A subscriber MAY request a subset of the advertised tracks.  A track
listed in `depends` that is not requested cannot be used to
reconstruct the dependent component ({{dependencies}}).  Alternate
encodings of one component share `altGroup` and are not shown
({{alternate-groups}}). A mesh presentation MAY advertise a
basemesh track together with either a geometry track, when that
track carries video-coded displacement, or an arithmetic-coded
displacement track.

### Packed Video Component Mode
{: #catalog-pack-attr}

{{fig-v3c-catalog-pack-attr}} shows a presentation in which attribute
data is carried in packed video and geometry remains its own
sub-bitstream.  The active VPS has geometry video and packed video
present.  It does not have separate occupancy or attribute video
sub-bitstreams, so those tracks are omitted ({{component-presence}}).
The geometry and packed tracks list the atlas track in `depends`.
Regions inside the packed frame, including which attributes they
carry, are recovered from the VPS packed-video extension specified by
{{ISOIEC23090-5}}.

~~~
{
  "version": "draft-01",
  "tracks": [
    {
      "namespace": "packed",
      "name": "v3c/atlas/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-atlas",
      "renderGroup": 1,
      "initRef": "atlas-init",
      "codec": "v3c1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "atlas",
        "unitType": 1,
        "atlasId": 0
      }
    },
    {
      "namespace": "packed",
      "name": "v3c/geometry/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-geometry",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 4000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "geometry",
        "unitType": 3,
        "atlasId": 0
      }
    },
    {
      "namespace": "packed",
      "name": "v3c/packed/0/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-packed",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 8000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "packed",
        "unitType": 5,
        "atlasId": 0,
        "packedMapIndex": 0
      }
    }
  ],
  "initDataList": [
    {
      "id": "atlas-init",
      "type": "inline",
      "data": "PLACEHOLDER-V3CDecoderConfigurationRecord"
    },
    {
      "id": "hevc-init",
      "type": "inline",
      "data": "PLACEHOLDER-HEVCDecoderConfigurationRecord"
    }
  ]
}
~~~
{: #fig-v3c-catalog-pack-attr title="Catalog when attributes are packed"}

### Packed Video Component Mode with Native Framing
{: #catalog-native}

{{fig-v3c-catalog-native}} shows the presentation of
{{catalog-pack-attr}} with native framing ({{native-framing}}).  The
atlas track references the entry `vps`, which contains the
`v3c_parameter_set()` of the active VPS.  The geometry and packed
tracks have no `initRef`; their HEVC parameter sets are carried in
band, and their `codec` value uses the sample entry type `hev1`
({{v3c-track-identifiers}}).  The atlas track `codec` value is
`v3cg`.  The `data` value is a placeholder.

~~~
{
  "version": "draft-01",
  "tracks": [
    {
      "namespace": "packed-native",
      "name": "v3c/atlas/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-atlas",
      "renderGroup": 1,
      "initRef": "vps",
      "codec": "v3cg",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "native",
        "component": "atlas",
        "unitType": 1,
        "atlasId": 0
      }
    },
    {
      "namespace": "packed-native",
      "name": "v3c/geometry/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-geometry",
      "renderGroup": 1,
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hev1",
      "bitrate": 4000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "native",
        "component": "geometry",
        "unitType": 3,
        "atlasId": 0
      }
    },
    {
      "namespace": "packed-native",
      "name": "v3c/packed/0/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-packed",
      "renderGroup": 1,
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hev1",
      "bitrate": 8000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "native",
        "component": "packed",
        "unitType": 5,
        "atlasId": 0,
        "packedMapIndex": 0
      }
    }
  ],
  "initDataList": [
    {
      "id": "vps",
      "type": "inline",
      "data": "PLACEHOLDER-v3c_parameter_set"
    }
  ]
}
~~~
{: #fig-v3c-catalog-native title="Catalog with native framing"}

### Unpacked Component Mode with Multiple Attributes
{: #catalog-pack-none}

{{fig-v3c-catalog-pack-none}} shows the same content with packed
video absent (`vps_packed_video_present_flag` equal to 0) and every
video component carried on its own track.  The active VPS has
occupancy video, geometry video, and attribute video present.  Each
attribute index is one AVD track.  For this example, the VPS maps the
attribute indices as follows:

* 0: texture (color)
* 1: transparency (opacity in Gaussian splat terminology)
* 2: scale
* 3: rotation
* 4: spherical harmonics

These are attribute indices, not the numeric `ai_attribute_type_id`
values.  The corresponding `ATTR_TEXTURE`, `ATTR_TRANSPARENCY`,
`ATTR_SCALE`, `ATTR_ROTATION`, and `ATTR_SPHERICAL_HARMONICS` types,
and the equivalence of transparency and Gaussian splat opacity, are
defined by {{ISOIEC23090-5}}.

Each video track lists `v3c/atlas/0` in `depends`.  `mapIndex` is
omitted because the VPS signals a single map
(`vps_map_count_minus1` equal to 0; {{map-streams}}).
`attributePartitionIndex` is 0 because none of these attributes is
split across partitions.

~~~
{
  "version": "draft-01",
  "tracks": [
    {
      "namespace": "unpacked",
      "name": "v3c/atlas/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-atlas",
      "renderGroup": 1,
      "initRef": "atlas-init",
      "codec": "v3c1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "atlas",
        "unitType": 1,
        "atlasId": 0
      }
    },
    {
      "namespace": "unpacked",
      "name": "v3c/occupancy/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-occupancy",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 500000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "occupancy",
        "unitType": 2,
        "atlasId": 0
      }
    },
    {
      "namespace": "unpacked",
      "name": "v3c/geometry/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-geometry",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 4000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "geometry",
        "unitType": 3,
        "atlasId": 0
      }
    },
    {
      "namespace": "unpacked",
      "name": "v3c/attribute/0/0/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 3000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 0,
        "attributePartitionIndex": 0
      }
    },
    {
      "namespace": "unpacked",
      "name": "v3c/attribute/0/1/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 1000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 1,
        "attributePartitionIndex": 0
      }
    },
    {
      "namespace": "unpacked",
      "name": "v3c/attribute/0/2/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 1000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 2,
        "attributePartitionIndex": 0
      }
    },
    {
      "namespace": "unpacked",
      "name": "v3c/attribute/0/3/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 1000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 3,
        "attributePartitionIndex": 0
      }
    },
    {
      "namespace": "unpacked",
      "name": "v3c/attribute/0/4/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 2000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 4,
        "attributePartitionIndex": 0
      }
    }
  ],
  "initDataList": [
    {
      "id": "atlas-init",
      "type": "inline",
      "data": "PLACEHOLDER-V3CDecoderConfigurationRecord"
    },
    {
      "id": "hevc-init",
      "type": "inline",
      "data": "PLACEHOLDER-HEVCDecoderConfigurationRecord"
    }
  ]
}
~~~
{: #fig-v3c-catalog-pack-none title="Catalog when every component is its own track"}

### Video-based Dynamic Mesh Coding (V-DMC)
{: #catalog-vdmc}

{{fig-v3c-catalog-vdmc}} shows a V-DMC presentation
({{ISOIEC23090-29}}) following the multi-track layout of
{{ISOIEC23090-10}}, Clause 7.6.3.  The presentation consists of an
atlas track, a basemesh track, an arithmetic-coded displacement
track, and a texture attribute video track.  The basemesh and
displacement tracks reference their own decoder configuration records
(`BaseMeshDecoderConfigurationRecord` and
`ACDisplacementDecoderConfigurationRecord`), and their `codec` values
are the corresponding sample entry types `bmc1` and `acd1`
({{v3c-track-identifiers}}).  The VPS in `atlas-init` carries a V-DMC
VPS extension and a `ptl_profile_toolset_idc` in the range reserved
for V-DMC.  When displacement is coded with a video codec instead,
{{ISOIEC23090-10}} carries it in a video component track with
`vuh_unit_type` equal to V3C_GVD.

~~~
{
  "version": "draft-01",
  "tracks": [
    {
      "namespace": "vdmc",
      "name": "v3c/atlas/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-atlas",
      "renderGroup": 1,
      "initRef": "atlas-init",
      "codec": "v3c1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "atlas",
        "unitType": 1,
        "atlasId": 0
      }
    },
    {
      "namespace": "vdmc",
      "name": "v3c/basemesh/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-basemesh",
      "renderGroup": 1,
      "initRef": "basemesh-init",
      "depends": ["v3c/atlas/0"],
      "codec": "bmc1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "basemesh",
        "unitType": 7,
        "atlasId": 0
      }
    },
    {
      "namespace": "vdmc",
      "name": "v3c/displacement/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-displacement",
      "renderGroup": 1,
      "initRef": "displacement-init",
      "depends": ["v3c/atlas/0", "v3c/basemesh/0"],
      "codec": "acd1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "displacement",
        "unitType": 8,
        "atlasId": 0
      }
    },
    {
      "namespace": "vdmc",
      "name": "v3c/attribute/0/0/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 3000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 0,
        "attributePartitionIndex": 0
      }
    }
  ],
  "initDataList": [
    {
      "id": "atlas-init",
      "type": "inline",
      "data": "PLACEHOLDER-V3CDecoderConfigurationRecord"
    },
    {
      "id": "basemesh-init",
      "type": "inline",
      "data": "PLACEHOLDER-BaseMeshDecoderConfigurationRecord"
    },
    {
      "id": "displacement-init",
      "type": "inline",
      "data": "PLACEHOLDER-ACDisplacementDecoderConfigurationRecord"
    },
    {
      "id": "hevc-init",
      "type": "inline",
      "data": "PLACEHOLDER-HEVCDecoderConfigurationRecord"
    }
  ]
}
~~~
{: #fig-v3c-catalog-vdmc title="Catalog for a V-DMC presentation"}

The displacement track lists the basemesh track in `depends` because
displacements are applied to the decoded basemesh during V-DMC
reconstruction ({{ISOIEC23090-29}}).
<!-- TODO: confirm with ISO/IEC 23090-29 whether the displacement sub-bitstream can be decoded without the basemesh sub-bitstream; if so, drop that dependency. -->

### Spatial Subdivision with Atlas Tile Tracks
{: #catalog-subdivision}

{{fig-v3c-catalog-tiles}} follows the layout of the multi-track
example with atlas tile tracks in {{ISOIEC23090-10}}, Annex D.4,
without an occupancy component.  The atlas frame is partitioned into
two atlas tiles with tile identifiers 1 and 2.  One atlas track
carries the atlas parameter sets and the non-ACL NAL units, and one
atlas tile track per tile carries the ACL NAL units of that tile
({{atlas-tiles}}).  For each tile, a geometry track and an attribute
track carry the video components of the corresponding region.  The
tile tracks use the `codec` value `v3t1` and reference the same
`initDataList` entry as the atlas track.

The two geometry tracks carry the same V3C unit header fields; they
are distinguished by the atlas tile track that each lists in
`depends`.  The same applies to the two attribute tracks.  The video
track names extend the recommended patterns with a `/tile/<tileId>`
suffix.  A subscriber interested in only one spatial region
subscribes to the atlas track, one atlas tile track, and the video
tracks that depend on that tile track.
<!-- TODO: decide whether v3c.tileId is also permitted on video component tracks to identify the tile region they carry. -->

~~~
{
  "version": "draft-01",
  "tracks": [
    {
      "namespace": "tiled",
      "name": "v3c/atlas/0",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-atlas",
      "renderGroup": 1,
      "initRef": "atlas-init",
      "codec": "v3c1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "atlas",
        "unitType": 1,
        "atlasId": 0
      }
    },
    {
      "namespace": "tiled",
      "name": "v3c/atlas/0/tile/1",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-atlas",
      "renderGroup": 1,
      "initRef": "atlas-init",
      "depends": ["v3c/atlas/0"],
      "codec": "v3t1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "atlas",
        "unitType": 1,
        "atlasId": 0,
        "tileId": 1
      }
    },
    {
      "namespace": "tiled",
      "name": "v3c/atlas/0/tile/2",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-atlas",
      "renderGroup": 1,
      "initRef": "atlas-init",
      "depends": ["v3c/atlas/0"],
      "codec": "v3t1",
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "atlas",
        "unitType": 1,
        "atlasId": 0,
        "tileId": 2
      }
    },
    {
      "namespace": "tiled",
      "name": "v3c/geometry/0/tile/1",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-geometry",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0", "v3c/atlas/0/tile/1"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 2000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "geometry",
        "unitType": 3,
        "atlasId": 0
      }
    },
    {
      "namespace": "tiled",
      "name": "v3c/attribute/0/0/0/tile/1",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0", "v3c/atlas/0/tile/1"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 1500000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 0,
        "attributePartitionIndex": 0
      }
    },
    {
      "namespace": "tiled",
      "name": "v3c/geometry/0/tile/2",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-geometry",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0", "v3c/atlas/0/tile/2"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 2000000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "geometry",
        "unitType": 3,
        "atlasId": 0
      }
    },
    {
      "namespace": "tiled",
      "name": "v3c/attribute/0/0/0/tile/2",
      "packaging": "v3c",
      "isLive": false,
      "role": "v3c-attribute",
      "renderGroup": 1,
      "initRef": "hevc-init",
      "depends": ["v3c/atlas/0", "v3c/atlas/0/tile/2"],
      "codec": "resv.vvvc.hvc1",
      "bitrate": 1500000,
      "framerate": 25,
      "timescale": 1000,
      "trackDuration": 2000,
      "template": [0, 40, [1, 0], [0, 1], 0, 0],
      "v3c": {
        "framing": "isobmff",
        "component": "attribute",
        "unitType": 4,
        "atlasId": 0,
        "attributeIndex": 0,
        "attributePartitionIndex": 0
      }
    }
  ],
  "initDataList": [
    {
      "id": "atlas-init",
      "type": "inline",
      "data": "PLACEHOLDER-V3CDecoderConfigurationRecord"
    },
    {
      "id": "hevc-init",
      "type": "inline",
      "data": "PLACEHOLDER-HEVCDecoderConfigurationRecord"
    }
  ]
}
~~~
{: #fig-v3c-catalog-tiles title="Catalog with atlas tile tracks"}

# Prioritization Considerations

MOQT supports prioritization of delivery under congestion through
subscriber priority and publisher priority
({{I-D.ietf-moq-transport-23}}, Section 5.1).  Priority values range from 0
to 255, where lower numeric values indicate higher priority.  Publisher
priority can be provided as a default track property
(`DEFAULT_PUBLISHER_PRIORITY`) and can also be set per subgroup or
datagram.

A V3C publisher SHOULD assign higher priority, using lower numeric
values, to components that are critical for reconstruction, such as
atlas and base geometry or basemesh.  Enhancement components, such as
high-resolution attributes or optional packed-video refinements,
SHOULD be assigned lower priority.  When `ptl_priority_order_enable_flag`
in the VPS is 1, {{ISOIEC23090-5}} defines a priority order among
video components: geometry, then attribute components in increasing
attribute index and partition index, then packed video components in
increasing packed map index.  A publisher SHOULD assign publisher
priorities to video component tracks that are consistent with that
order.  Single-track mode ({{single-track}}) does not allow
per-component prioritization.  In datagram mode,
reconstruction-critical datagrams SHOULD be prioritized over optional
enhancement datagrams.

Subscribers MAY adjust priorities dynamically based on viewport,
bandwidth, rendering requirements, or selected alternatives, including
by sending REQUEST_UPDATE.  For example, if V3C tiles are exposed as
separate tracks, subgroups, or datagram objects, a subscriber can
prioritize tiles in or near the current viewport and deprioritize
less relevant tiles.

# Relationship to the RTP Payload Format for V3C

{{RFC10034}} defines an RTP payload format for V3C atlas
sub-bitstreams and uses SDP to carry the VPS and V3C unit-header
identity.  RTP payload formats for V3C video sub-bitstreams are
defined by the relevant RTP payload formats for the applicable video
codecs.

This document provides the corresponding functions with MSF catalog
fields rather than SDP:

* the initialization data referenced by the atlas track `initRef`
  replaces `sprop-v3c-parameter-set`.  With native framing, it
  carries the same `v3c_parameter_set()` value.  With ISOBMFF
  framing, it is a `V3CDecoderConfigurationRecord`, whose setup unit
  arrays correspond to `sprop-v3c-atlas-data`.  The VPS remains the
  source of attribute type, presence flags, and PTL, as in SDP;
* the catalog `v3c` object replaces `sprop-v3c-unit-type`,
  `sprop-v3c-atlas-id`, `sprop-v3c-attr-idx`,
  `sprop-v3c-attr-part-idx`, `sprop-v3c-map-idx`,
  `sprop-v3c-aux-video-flag`, and related parameters.  As with
  `sprop-v3c-attr-idx`, `attributeIndex` is an index; the type is in
  the VPS ({{vps-attribute-types}}).  `auxiliaryVideo` replaces
  `sprop-v3c-aux-video-flag`.  `parameterSetId` replaces
  `sprop-v3c-vps-id`.  `packedMapIndex` has no SDP
  counterpart in {{RFC10034}} because that RFC's informative unit
  header treated packed-video bits as reserved;
* `tileId`, when used, replaces `sprop-v3c-tile-id` for an optional
  per-tile atlas track ({{atlas-tiles}});
* `renderGroup` and `depends` replace the RTP grouping mechanism that
  identifies streams of one V3C representation ({{RFC10034}},
  Section 9.3);
* MOQT Groups replace RTP timestamp alignment for the V3C composition
  unit.

RTP-style NAL fragmentation across packets is not reused.  Oversized
coded units use MOQT subgroup streams ({{objects-subgroups-datagrams}}).
When datagram delivery is used, slice and NAL unit size SHOULD be constrained
at encode time rather than by fragmenting NAL units across datagrams
({{datagram-encoding}}).

# Security Considerations

This document inherits the security considerations of MOQT
{{I-D.ietf-moq-transport-23}}, QUIC {{RFC9000}}, and MSF
{{I-D.ietf-moq-msf}}.  In particular, publishers and relays must
consider resource exhaustion attacks caused by large numbers of
tracks, objects, subgroups, or datagrams, and must validate bounds
before allocating memory.

Authentication, authorization, and confidentiality of MOQT sessions
are provided by the underlying QUIC or WebTransport mapping.  Object
payloads remain opaque to relays.  When end-to-end confidentiality
and integrity of media are required, deployments can use the MSF
content-protection mechanisms, including MoQ Secure Objects
{{I-D.ietf-moq-secure-objects}} as referenced by {{I-D.ietf-moq-msf}}.
The VPS and any component decoder configuration records in
`initDataList` are catalog payload.  An attacker that tampers with
them can cause decoder misconfiguration or reconstruction failure.  Catalog objects
and initialization data MUST be protected with at least the same
integrity guarantees as the component media they initialize.

Before decoding, a receiver MUST verify that the catalog is
consistent: every `depends` entry and `initRef` value resolves to a
track or `initDataList` entry advertised in the same catalog and
namespace, and the `v3c` identifiers of each track agree with the
components that the VPS signals as present.  Accepting a substituted
atlas track or VPS is a cross-component integrity failure: geometry
might be reconstructed against attacker-chosen patch metadata.

Datagram delivery can increase the rate of object loss under
congestion and can cause receivers to observe incomplete sets of V3C
components for a given volumetric frame.  Implementations SHOULD
validate component dependencies before decoding or rendering and
SHOULD tolerate missing optional datagram-delivered objects.
Incomplete geometry, packed video, or atlas data MUST NOT be treated as a complete
volumetric frame.

Replay of an old Group under a current VPS, or of a current Group
under a stale VPS, can produce incorrect reconstruction.  Subscribers
SHOULD bind decoded Groups to the catalog generation that advertised
their `initRef`.

A downgrade to packaging `loc` on a V3C component track would strip
V3C identity from the catalog.  A subscriber MUST NOT decode a track
as a V3C component unless its `packaging` is `v3c`; this includes
tracks that carry a V3C role ({{track-roles}}) or a `v3c` object
with another packaging value.

Relay-visible track and object metadata, including track names,
priorities, and subscription filters, can
reveal information about scene structure or user interest (for
example, which attribute or viewport-related tracks are requested).
Deployments that consider this sensitive should avoid placing viewport
or tile identity in relay-visible properties and should consider
end-to-end object protection.

# IANA Considerations
{: #iana}

This document has no IANA actions at this time.
{{I-D.ietf-moq-msf}} does not currently establish IANA registries for
catalog packaging values or track roles.  This document defines `v3c`
as an extension packaging value ({{v3c-packaging-type}}) and uses the
custom role values defined in {{track-roles}}, as permitted by MSF.
If MSF establishes registries for packaging values or track roles in
a future revision, registration of these values can be requested at
that time.

--- back

# Design Options for Datagram-Mode Encoding
{: #datagram-options}

This appendix is informative.  It does not change
{{datagram-encoding}}.  It states the design problem, the working
choice in this document, and alternatives on which Media Over QUIC
Working Group input is requested.

## Problem

{{I-D.ietf-moq-transport-23}}, Section 11.2, maps one MOQT object with
Delivery Mode Datagram to one OBJECT_DATAGRAM.  If the
object header, object properties, and payload exceed the session
maximum datagram size, the object is dropped without notification.
Each hop can have a different maximum.  Relays can add Object
Properties and thereby increase the encoded size.

QUIC DATAGRAM frames cannot be fragmented {{RFC9221}}.  WebTransport
exposes a smaller application datagram size than the QUIC
`max_datagram_frame_size` because HTTP Datagram framing consumes part
of the frame {{I-D.ietf-webtrans-http3}}.

A coded V3C unit is typically one access unit and may contain several
NAL units.  Atlas NAL units have no codec-imposed maximum size
({{RFC10034}}, Section 5.4.1).  Video access units commonly exceed a
path MTU.  RTP therefore defines Single NAL Unit, Aggregation Packet,
and Fragmentation Unit structures ({{RFC10034}}, Section 5.4).  MOQT
does not provide an equivalent datagram fragment.

The mapping question is therefore: when Datagram delivery is used,
what is the independently transmissible unit, and who is responsible
for keeping that unit within the datagram budget?

## Option A: Encoder-Constrained Coded Unit (Working Specification)

One MOQT Object remains one coded V3C unit.  The encoder is
configured so that each slice, NAL unit, and the resulting coded unit
fit in one OBJECT_DATAGRAM after packaging.  If they do not, the
publisher uses subgroup stream delivery for that object.

~~~
Coded V3C unit (access unit)
        |
        |  encode to datagram budget
        v
One MOQT Object  --->  one QUIC or WebTransport datagram
        |
        |  otherwise
        v
Subgroup stream
~~~
{: #fig-option-a title="Option A: encoder-constrained coded unit"}

Pros:

* Relays stay unaware of V3C and NAL syntax.
* The object mapping in {{mapping}} is unchanged for both delivery
  modes.
* No V3C datagram fragment header is defined.

Cons:

* The encoder must know a transport size that is not available until
  the MOQT session exists, and that can differ on later hops.
* Constraining an entire access unit to a datagram often forces many
  small slices, which can reduce coding efficiency.
* A publisher cannot send a large independently useful unit as a
  datagram without either re-encoding or switching to a stream.

This document currently specifies Option A.

## Option B: NAL Unit or Slice as Datagram Object

When Delivery Mode is Datagram, each independently
packetizable NAL unit or coded slice is one MOQT Object.  Object IDs
increase in decoding order within the Group.  Small NAL units MAY
share one object when they fit.  Subgroup delivery can keep the
coded-unit mapping of {{objects-subgroups-datagrams}}.

~~~
Access unit = NAL 0 + NAL 1 + NAL 2
        |
        v
Object i      NAL 0     DATAGRAM
Object i+1    NAL 1     DATAGRAM
Object i+2    NAL 2     DATAGRAM
~~~
{: #fig-option-b title="Option B: NAL or slice as datagram object"}

Pros:

* Matches RTP Single NAL Unit packetization more closely than Option A
  ({{RFC10034}}, Section 5.4.2).
* Loss of one datagram drops one NAL unit, not necessarily the
  entire coded unit.  Atlas tiles that are already independently useful
  benefit.
* Slice size is an existing encoder control; the packager does not
  invent a fragment header.

Cons:

* The datagram object boundary is no longer the coded V3C unit.  A
  subscriber must reassemble an access unit from consecutive objects
  before some video decoders can consume it.
* Publishers would emit one object per NAL unit for datagram
  tracks, which {{objects-subgroups-datagrams}} currently advises
  against when the only goal is codec framing.
* An access unit that still contains one oversized NAL unit cannot be
  sent as datagrams without also adopting Option C or D, or
  re-encoding.

## Option C: Application-Level Fragments of One Coded Unit

The object mapping stays "one coded V3C unit".  The packager splits
that unit across several OBJECT_DATAGRAM payloads.  Each payload
begins with a streaming-format fragment header, for example a fragment
index and a fragment count.  Fragments are distinct MOQT objects
(consecutive Object IDs).  Using the same Object ID for several
datagrams would require a change to MOQT and is not proposed here.

~~~
Coded V3C unit  (one access unit)
        |
        v
Object k     fragment 0 of N     DATAGRAM
Object k+1   fragment 1 of N     DATAGRAM
Object k+2   fragment 2 of N     DATAGRAM
~~~
{: #fig-option-c title="Option C: application-level fragments"}

Pros:

* The encoder need not target a datagram MTU.
* Relays still see opaque objects.
* Subgroup delivery can carry the same unit unfragmented.

Cons:

* This document would have to define a fragment header, reassembly,
  and loss behavior.  Loss of any fragment typically makes the coded
  unit useless, which is weaker than a subgroup stream and similar to
  IP fragmentation.
* Receivers must wait for N datagrams and detect missing fragments
  without MOQT-level notification.
* The same problem exists for LOC samples and CMAF chunks.  Defining
  fragments only in the V3C packaging may be the wrong layer.

## Option D: RFC 10034 RTP Payload Format for V3C

Object payloads would use the RFC 10034 structures: Single NAL Unit,
Aggregation Packet, and Fragmentation Unit.  Each RTP-sized payload
would be one MOQT Object sent as a datagram.

Pros:

* Reuses an existing V3C atlas packetizer, including FU without
  encoder cooperation ({{RFC10034}}, Section 5.4.4).
* Aggregation of small NAL units is already specified.

Cons:

* RTP payload headers, decoding-order numbers, and FU headers would
  appear inside MOQT objects.  Sequencing would mix RTP and MOQT
  object identities.
* Video components would still need the corresponding RTP payload
  formats, not only RFC 10034.
* This option conflicts with the current rule that this document
  does not define RTP packetization inside MOQT objects.

## Option E: Datagrams Only for Units That Already Fit

Do not try to place ordinary video or atlas access units in datagrams.
Use Delivery Mode Datagram only for units that are
already small and independently useful, for example viewport tiles,
optional enhancement data, or small atlas NAL units.  Use subgroup
streams for reconstruction-critical and oversized coded units.

~~~
Track (mixed delivery modes)
Group n
    Subgroup 0     base geometry AU     stream
    Datagram       optional tile / LoD  DATAGRAM
~~~
{: #fig-option-e title="Option E: datagrams only when the unit already fits"}

Pros:

* Requires no new header and no encoder-to-MTU coupling beyond "do
  not datagram what does not fit".
* Matches MOQT's existing drop-if-too-large rule and the permission
  to mix Subgroup and Datagram in one track
  ({{I-D.ietf-moq-transport-23}}, Section 2.1).
* Reconstruction-critical data keeps reliable ordered delivery.

Cons:

* Geometry and occupancy access units stay on streams, so this
  option does not provide datagram latency for those components.
* Publishers need a clear policy for which V3C unit types are
  eligible.  That policy could live in this document without new wire
  formats.

Option E can be combined with Option A or B for the units that remain
eligible for datagrams.

## Orthogonal Questions

These questions apply to more than one option.

Scope:
: Is datagram size a V3C packaging issue, a general MSF/LOC issue,
  or a MOQT issue?  LOC samples and other MSF packagings face the same
  OBJECT_DATAGRAM limit.

Budget signaling:
: Should the encoder learn the budget only from the local session
  (`max_datagram_frame_size` or the WebTransport maximum datagram
  size), or should MSF advertise a conservative object-payload
  budget in the catalog so encoding can start before SETUP?

Path heterogeneity:
: The original publisher knows only the first hop.  Should a relay
  that cannot forward an OBJECT_DATAGRAM on a smaller downstream
  session drop it (current MOQT), or must the publisher leave
  headroom?  MOQT version 23 requires an object in a subscription to
  be sent according to its Delivery Mode, so a relay cannot rewrite
  Datagram delivery to a subgroup stream.

Relay inflation:
: How much headroom should a publisher reserve for Object Properties
  added downstream?  This document currently leaves that as a
  publisher choice.

Fallback:
: MOQT already allows mixing Subgroup and Datagram in one track.
  Is per-object fallback from Datagram to Subgroup when the encoded
  unit is too large (Option A) sufficient, or must a track declare a
  single delivery mode?

## Requested WG Input to Datagram Design Options 

The authors request comment on:

1. Whether Option A should remain the V3C mapping for datagrams.
2. Whether Option B should be permitted or required when Datagram
   delivery is used, including for atlas tiles.
3. Whether Option C belongs in this document, in MSF, or not at
   all.
4. Whether Option D (RTP structures inside MOQT objects) should stay
   out of scope.
5. Whether Option E should be the recommended operational profile,
   with datagrams limited to independently discardable units that
   already fit.
6. Which layer should specify the datagram payload budget and any
   headroom for relays.

# Single-Track Encapsulation Mode
{: #single-track}

This appendix specifies single-track mode, in which all V3C
components are carried in one MOQT track.  This mode reduces the
number of subscriptions, but it prevents per-component subscription,
prioritization, and independent adaptive bitrate switching, and
therefore is NOT RECOMMENDED.  A publisher that uses single-track
mode MUST follow this appendix.  Except as specified in this
appendix, the requirements in the body of this document apply.

## Track and Catalog

Each Object carries one complete V3C composition unit as a sequence
of V3C units, each preceded by a length field and each including its
four-byte V3C unit header.  Because every Object carries all
components of one composition unit, subgroups cannot separate
component types.

The catalog `v3c` object of the track omits all fields except
`parameterSetId` and `framing` ({{v3c-track-identifiers}}).  The
`codec` value is `v3e1` or `v3eg` for V-PCC, or `vdm1` or `vdmg` for
V-DMC ({{ISOIEC23090-10}}, Clauses 7.4.1 and 7.6.2).  With native
framing, the `codec` value is `v3eg` or `vdmg`.

## Initialization Data

The track references with `initRef` the VPS as specified in
{{initialization}}: the `v3c_parameter_set()` with native framing, or
a `V3CDecoderConfigurationRecord` with ISOBMFF framing.  A V3C unit
of type V3C_VPS MAY also be included in the first Object of a Group
so that a stored Group remains self-contained.

With ISOBMFF framing, carriage of video decoder configuration for
this mode is not specified by this document.

<!-- TODO: specify video decoder configuration for single-track mode with ISOBMFF framing, or restrict single-track mode to native framing. -->

## Object Payload

With native framing, an Object payload is a V3C sample stream
({{ISOIEC23090-5}}, Annex C): one `sample_stream_v3c_header()`
followed by one `sample_stream_v3c_unit()` for each V3C unit of the
composition unit.  Each `sample_stream_v3c_unit()` contains the
complete `v3c_unit()`, including its V3C unit header.  The size of
each `ssvu_v3c_unit_size` field is given by
`ssvh_unit_size_precision_bytes_minus1` of that Object's header.

With ISOBMFF framing, an Object payload is a `V3CBitstreamSample`
({{ISOIEC23090-10}}, Clause 7.3.1.2).  The size of its length fields
is given by `unit_size_precision_bytes_minus1` in the
`V3CDecoderConfigurationRecord` that the track references with
`initRef`.

# Acknowledgments
{:numbered="false"}

The authors thank contributors to the Media Over QUIC Working Group
and to the V3C RTP payload format work for discussion of volumetric
media carriage.
