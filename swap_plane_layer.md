import pcbnew

board = pcbnew.GetBoard()
gnd_keys = ["GND","AGND","PGND"]
power_keys = ["VDD","VBAT","VIN","3V3","1V8"]

changed = 0
for z in board.Zones():
    netname = str(z.GetNetname()).upper()
    if any(k in netname for k in gnd_keys):
        if z.GetLayer() != pcbnew.In1_Cu:   # Layer 2
            z.SetLayer(pcbnew.In1_Cu)
            changed += 1
    elif any(k in netname for k in power_keys):
        if z.GetLayer() != pcbnew.In2_Cu:   # Layer 3
            z.SetLayer(pcbnew.In2_Cu)
            changed += 1

try:
    pcbnew.ZONE_FILLER(board).Fill(board.Zones())
except Exception:
    pass

pcbnew.Refresh()
print(f"✅ Reassigned {changed} zones: Layer2=GND, Layer3=Power")
