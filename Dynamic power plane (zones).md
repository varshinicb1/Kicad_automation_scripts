import pcbnew

board = pcbnew.GetBoard()
def mm(v): return pcbnew.FromMM(v)

nets = ["VIN", "VDD_3V3", "VBAT"]

for netname in nets:
    net = board.FindNet(netname)
    if not net:
        print(f"⚠️ Net {netname} not found.")
        continue

    # Collect all pads belonging to this net
    pads = []
    for fp in board.GetFootprints():
        for pad in fp.Pads():
            if pad.GetNetname() == netname:
                pads.append(pad)

    if not pads:
        print(f"⚠️ No pads found for {netname}")
        continue

    # Make one zone per cluster of pads (here we just make 1 covering all)
    zone = pcbnew.ZONE(board)
    zone.SetLayer(pcbnew.In1_Cu)  # Layer 2 = Power
    zone.SetNetCode(net.GetNetCode())

    poly = zone.Outline()
    poly.NewOutline()

    # Bounding box around all pads in this net
    xs = [p.GetPosition().x for p in pads]
    ys = [p.GetPosition().y for p in pads]
    margin = mm(2.0)

    x1, x2 = min(xs)-margin, max(xs)+margin
    y1, y2 = min(ys)-margin, max(ys)+margin

    poly.Append(x1, y1)
    poly.Append(x2, y1)
    poly.Append(x2, y2)
    poly.Append(x1, y2)
    poly.Append(x1, y1)

    board.Add(zone)
    print(f"✅ Dynamic power island created for {netname}")

0
1
2
3
4
5
✅ Dynamic power island created for VIN
⚠️ No pads found for VDD_3V3
0
1
2
3
4
5
✅ Dynamic power island created for VBAT
# Fill all zones
filler = pcbnew.ZONE_FILLER(board)
filler.Fill(board.Zones())
True
pcbnew.Refresh()

