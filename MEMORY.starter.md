# Sage/Waggle starter memory (sanitized — safe to share)
#
# These are generally-true, non-sensitive facts about the Sage/Waggle platform,
# distilled from real plugin-development experience. They contain NO credentials,
# phone numbers, private emails, tokens, or node-specific secrets. Seed these into
# your own ~/.hermes/memories/MEMORY.md if useful. Keep memory compact — it's
# injected every turn and has a small char budget. Load the `sage-waggle` skill
# for the full detail behind each of these.

Sage plugin pods do NOT self-identify: /etc/waggle is a node-HOST path, NOT mounted into pods. A pod sees only WAGGLE_PLUGIN_* env + /run/waggle/{uploads,data-config.json}. Beehive attaches vsn/gps DOWNSTREAM via message routing, so a plugin does not need to know its own node to upload correctly.

pywaggle upload_file(timestamp=capture_ts) overrides get_timestamp; upload key = <ts>-<sha1>; meta values are ALL strings. Never fabricate node identity or coordinates — omit when unknown.

Sage ECR "Register & Build": the fleet-wide runc /proc/acpi failure (CVE-2025-31133) was FIXED in 2026-07 — CPU/slim-base plugins now build fine via the portal Register-and-Build path. GPU/CUDA plugins (nvcr.io/nvidia base) STILL fail ECR (a separate QEMU cross-build crash), so for those the working deploy is: build natively on the ARM64 node + side-load into k3s with pluginctl (no ECR). GPU pluginctl pods must set --resource limit.memory=16Gi or they OOMKill (137); a -v volume mount requires --selector zone=core.

Cache-mediated cascade: chain vision plugins with NO cross-plugin calls by using the shared /local-cache as the only coupling. A detector (sage-yolo2) can ALSO be a producer — it crops each detection and writes the crop into a NEW cache stream as a byte-compatible v2 frame; a classifier (sage-bioclip2) consumes those crops on its own schedule. Because raw frames and crops are the identical v2 format, the classifier's source is a single --input path parameter (point at the raw stream for full frames, or the crop stream for crops) — no code change. Crop metadata carries a source{} provenance block so a species result traces back to the detection + parent frame + bbox. Everything stays frame-anchored (observation time = capture time).

WSN nodes have a GPS receiver (Geekstory VK-162, u-blox 7) and run gpsd inside a gps-server pod on ClusterIP :2947 (in-cluster DNS: wes-gps-server.default.svc.cluster.local:2947). gpsd yields position only (lat/lon/alt/mode/time + sats/HDOP), never node identity. A static pole-mounted node still emits a live, slightly-jittering fix — deployment-mobility and GPS-fix-liveness are independent.

node-manifest-v2.json (on the node HOST) has gps_lat/gps_lon (may be null = unsurveyed) and node hardware. Plugins default lat/lon to a sentinel then read the manifest when running on-node; fail gracefully (geo-filtering disabled) when the manifest is absent (dev machines).

BirdNET does NOT normalize input amplitude — faint audio scores low. Pre-amplify captures with a MEASURED FIXED gain (ffmpeg volume=NdB), not dynaudnorm/loudnorm (those compress dynamic range and hurt SNR). Expose all model params (bandpass_fmin/fmax matter for bandwidth-limited camera mics).

Reolink cameras: FLV/BCS auth uses query params (&user=&password=), NOT HTTP basic auth (basic-auth form makes ffmpeg fail with exit 187). Mobotix M16 MxPEG DOES use basic auth. Single-quote camera URLs in shell (passwords with ! trigger history expansion). Never put a real camera credential in a skill/repo/argv — pass via env/Secret and get the value from the node owner.

Camera metadata does NOT live in the RTSP video stream — acquire a native still (best metadata) and fall back to decode-from-H.264 only as a floor. Strong bias against re-encoding images.
