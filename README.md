# EmrB-Docking
cd ~/Documents/docking
# keep only protein atoms (remove waters/hetero)
cp emrb.pdb emrb_raw.pdb
awk 'substr($0,1,4)=="ATOM" || $1=="TER" || $1=="END"' emrb_raw.pdb > emrb_clean.pdb
# repair missing bits + add H at pH 7.4 (Python PDBFixer)
python3 - <<'PY'
from pdbfixer import PDBFixer
from openmm.app import PDBFile
fixer = PDBFixer(filename='emrb_clean.pdb')
fixer.findMissingResidues()
fixer.findMissingAtoms()
fixer.addMissingAtoms()
fixer.addMissingHydrogens(7.4)
with open('emrb_fixed.pdb','w') as f:
    PDBFile.writeFile(fixer.topology, fixer.positions, f)
print("Wrote emrb_fixed.pdb")
PY
# convert to receptor PDBQT
obabel emrb_fixed.pdb -O emrb.pdbqt -xh --partialcharge gasteiger
# If Vina reports Unknown or inappropriate tag … ROOT/BRANCH/TORSDOF:
awk '!($1 ~ /^(BRANCH|ENDBRANCH|ROOT|ENDROOT|TORSDOF)/)' emrb.pdbqt > emrb_tmp.pdbqt && mv emrb_tmp.pdbqt emrb.pdbqt
obabel cccp_3d.sdf -O cccp_74.pdbqt -p 7.4 -xh --partialcharge gasteiger
vina \
  --receptor emrb.pdbqt \
  --ligand cccp_74.pdbqt \
  --center_x 217.834 --center_y 222.829 --center_z 314.627 \
  --size_x 20 --size_y 20 --size_z 20 \
  --exhaustiveness 24 --num_modes 20 --energy_range 4 --seed 42 \
  --out cccp_emrb_out.pdbqt
vina_split --input cccp_emrb_out.pdbqt
# note: files are named with leading zeros
obabel -ipdbqt cccp_emrb_out_ligand_01.pdbqt -opdb -O cccp_74_top1.pdb
