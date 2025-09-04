import pcbnew

board = pcbnew.GetBoard()
mm = pcbnew.FromMM

for fp in board.GetFootprints():
    ref = fp.Reference()
    if not ref: 
        continue

    # Current size
    h = ref.GetTextHeight()
    w = ref.GetTextWidth()
    th = ref.GetTextThickness()

    # Scale down by 40%
    ref.SetTextHeight(int(h * 0.6))
    ref.SetTextWidth(int(w * 0.6))
    ref.SetTextThickness(int(th * 0.6))

pcbnew.Refresh()
print("✅ Silkscreen reference designators reduced by 40%")
