import pcbnew

board = pcbnew.GetBoard()
def mm(v): return pcbnew.FromMM(v)

nets = ["VIN", "VDD_3V3", "VBAT"]
margin = mm(1.0)   # extra copper around pad

# Remove existing zones for these nets
to_remove = []
for z in board.Zones():
    try:
        if z.GetNetname() in nets:
            to_remove.append(z)
    except:
        pass
for z in to_remove:
    board.Remove(z)

for netname in nets:
    net = board.FindNet(netname)
    if not net:
        print(f"⚠️ Net {netname} not found, skipping")
        continue

    for fp in board.GetFootprints():
        for pad in fp.Pads():
            if pad.GetNetname() != netname:
                continue

            # Pad bounding box
            bb = pad.GetBoundingBox()
            x1, y1 = bb.GetX()-margin, bb.GetY()-margin
            x2, y2 = bb.GetX()+bb.GetWidth()+margin, bb.GetY()+bb.GetHeight()+margin

            # Create a zone just around this pad
            zone = pcbnew.ZONE(board)
            zone.SetLayer(pcbnew.In1_Cu)   # Layer 2
            zone.SetNetCode(net.GetNetCode())

            poly = zone.Outline()
            poly.NewOutline()
            poly.Append(x1, y1)
            poly.Append(x2, y1)
            poly.Append(x2, y2)
            poly.Append(x1, y2)
            poly.Append(x1, y1)

            board.Add(zone)
            print(f"✅ Mini-zone created for {netname} pad {pad.GetPadName()} ({fp.GetReference()})")

# Fill all zones
filler = pcbnew.ZONE_FILLER(board)
filler.Fill(board.Zones())
pcbnew.Refresh()
print("🔄 Per-pad power islands regenerated & refilled.")
