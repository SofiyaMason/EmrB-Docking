cd ~/Documents/docking
obabel cccp.sdf -O cccp_3d.sdf --gen3d --conformer --nconf 40 --score rmsd --cutoff 0.5 --writeconformers
obabel cccp_3d.sdf -O cccp_8.pdbqt -p 8.0 -xh --partialcharge gasteiger

 vina \
  --receptor emrb.pdbqt \
  --ligand cccp_8.pdbqt \ 
  --center_x 217.834 --center_y 222.829 --center_z 314.627 \
  --size_x 20 --size_y 20 --size_z 20 \
  --exhaustiveness 24 --num_modes 20 --energy_range 4 --seed 42 \
  --out cccp8_emrb_out.pdbqt

vina_split --input cccp8_emrb_out.pdbqt
obabel -ipdbqt cccp8_emrb_out_ligand_01.pdbqt -opdb -O cccp_8_top1.pdb
