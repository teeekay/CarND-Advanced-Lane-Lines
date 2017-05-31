H range for lane lines values 10-3?



signs_vehicles_xygrad (white lines)
h - 15 - 22
s - 10-23 (lots of interference in road)
v - 220-256
H - 14-17 
S - 252-255 + 120-140 + 20-25
L - 225-250

straight_lines1.jpg Yellow and white lines
h - 18-23 (yellow) 11-21 (white
s - 0-12 (white) 140-230 (yellow)
v - 210+ (both - good)
H - 15-22 (Both! 21-22 yellow, 16-white)
S - 228-255(both!) 110-160(yellow
L - 220-255 (white), 140-180 (yellow)

test1 (yellow + white noisy)
combined sobel - weak
h 17-31 both moderate
s 145-220, 0-20
v 225-255 both - good
H 19-24 (yellow) 24-30 (white)
S 220-255 (both) 100-220 edges
L 220+(white) 150-160 (yellow but not separated from background)

test2 (yellow + white noisy)
combined sobel - good
h 17-25 (yellow good - white poor)
s 140-240(yellow), can't separate white
v 225-255 both - good
H 14-23 (yellow) 14-30 (both -white moderate)
S 220-255 (both) 100-220 edges
L 220+(white) 140-180 (yellow) ->140-255 (both good)

test3 (yellow + white noisy)
combined sobel - good
h 15-30 (yellow ok, white -weak)
s 0-12 white (moderate) 140-240 (yellow moderate)
v 225+ good both
H 14-30 yellow good - white moderate
S 100+ good
L 140+ good

test4 - both and shadows
combined sobel weak
h 17-30 yellow good (white can't separate?
s 0-12 (white noisy?)  140-240 (yellow weak?)
v 225+ both good?
H 16-30 both noisy
S 100+ good but picks up shdow - 190+ better (weak in shadow)
L 140-180 yellow picks up noise in road+ ? 220+ white good

test5 - both and shadows and 2 road surfaces
h -yellow good white poor
s- yellow good - no white
v both good until road changes
H - mess
S - weak
S2 - good but shadow
L - White good
L2 - not good

test6
combined sobel good
h - yellow good white weak
s - yellow no white
v - both good
H - both - rough
S - both yellow moderate white weak
S2 - both weak
L - white no yellow
L2 - weak

v