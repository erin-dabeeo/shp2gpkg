import os
import re
import geopandas as gpd

# 1. 작업할 루트 폴더 경로 지정 (현재 스크립트 위치 기준이면 '.' 사용)
root_dir = r"./your_root_folder_path" 

# 2. '13_숫자_숫자' 패턴을 매칭하기 위한 정규식 패턴 컴파일
# 예: "13_1234_5678" -> x="1234", y="5678" 추출
folder_pattern = re.compile(r"^13_(\d+)_(\d+)$")

print("GeoPackage 변환 작업을 시작합니다...\n")

# 루트 폴더 바로 밑의 자식 폴더들만 탐색
for child_dir in os.listdir(root_dir):
    child_path = os.path.join(root_dir, child_dir)
    
    # 폴더인지 확인하고 정규식 패턴과 일치하는지 체크
    if os.path.isdir(child_path):
        match = folder_pattern.match(child_dir)
        
        if match:
            x_index = match.group(1)
            y_index = match.group(2)
            
            # 예상되는 shp 파일 경로 조립
            shp_path = os.path.join(child_path, "road", "road.shp")
            
            # 해당 경로에 실제 shp 파일이 존재하는지 확인
            if os.path.exists(shp_path):
                # 저장할 GPKG 파일명 정의 (예: 13_1234_5678.gpkg)
                gpkg_filename = f"13_{x_index}_{y_index}.gpkg"
                gpkg_path = os.path.join(root_dir, gpkg_filename)
                
                try:
                    print(f"변환 중: {shp_path} -> {gpkg_filename}")
                    
                    # Shapefile 읽기
                    gdf = gpd.read_file(shp_path)
                    
                    # GeoPackage로 저장 (layer 이름은 기본값이나 파일명 등으로 지정 가능)
                    gdf.to_file(gpkg_path, driver="GPKG", layer="road")
                    
                except Exception as e:
                    print(f"❌ 변환 실패 ({child_dir}): {e}")
            else:
                print(f"⚠️ 파일을 찾을 수 없음 (스킵): {shp_path}")

print("\n모든 변환 작업이 완료되었습니다!")
