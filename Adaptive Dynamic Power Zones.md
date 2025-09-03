import pcbnew

board = pcbnew.GetBoard()
def mm(v): return pcbnew.FromMM(v)

nets = ["VIN", "VDD_3V3", "VBAT"]
margin = mm(2.0)   # margin around pad clusters

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

# Board outline box
bbox = board.GetBoardEdgesBoundingBox()
bx1, by1 = bbox.GetX(), bbox.GetY()
bx2, by2 = bx1 + bbox.GetWidth(), by1 + bbox.GetHeight()

for netname in nets:
    net = board.FindNet(netname)
    if not net:
        print(f"⚠️ Net {netname} not found, skipping")
        continue

    # collect all pads
    xs, ys = [], []
    for fp in board.GetFootprints():
        for pad in fp.Pads():
            if pad.GetNetname() == netname:
                pos = pad.GetPosition()
                bb = pad.GetBoundingBox()
                xs += [bb.GetX(), bb.GetX() + bb.GetWidth()]
                ys += [bb.GetY(), bb.GetY() + bb.GetHeight()]

    if not xs:
        print(f"⚠️ No pads for {netname}, skipping")
        continue

    # bounding box around pads
    x1, x2 = min(xs)-margin, max(xs)+margin
    y1, y2 = min(ys)-margin, max(ys)+margin

    # clip to board outline
    x1, y1 = max(x1, bx1), max(y1, by1)
    x2, y2 = min(x2, bx2), min(y2, by2)

    zone = pcbnew.ZONE(board)
    zone.SetLayer(pcbnew.In1_Cu)  # Layer 2
    zone.SetNetCode(net.GetNetCode())

    poly = zone.Outline()
    poly.NewOutline()
    poly.Append(x1, y1)
    poly.Append(x2, y1)
    poly.Append(x2, y2)
    poly.Append(x1, y2)
    poly.Append(x1, y1)

    board.Add(zone)
    print(f"✅ Zone shaped around {netname} pads")

# Refill all zones
filler = pcbnew.ZONE_FILLER(board)
filler.Fill(board.Zones())
pcbnew.Refresh()
print("🔄 Dynamic adaptive power planes regenerated & refilled.")
