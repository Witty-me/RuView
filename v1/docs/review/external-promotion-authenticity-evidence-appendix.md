# 附件：E-01 ~ E-10 证据摘录（代码与原文）

> 用途：作为《竞品外宣真实性调研报告（技术证据版）》的证据附件。
> 
> 说明：以下内容按 E-01 ~ E-10 编号整理，均来自仓库当前文件原文摘录。

---

## E-01 路由器 CSI 解析未实现（代码证据）

**来源**：`v1/src/hardware/router_interface.py`（约 L201-L223）

```python
    def _parse_csi_response(self, response: str) -> CSIData:
        ...
        raise RouterConnectionError(
            "Real CSI data parsing from router responses is not yet implemented. "
            "Collecting CSI data from a router requires: "
            "(1) a router with CSI-capable firmware ..."
        )
```

**证据要点**：核心 CSI 响应解析函数明确声明“尚未实现”。

---

## E-02 硬件连接生命周期为占位实现（代码证据）

**来源**：`v1/src/hardware/csi_extractor.py`（约 L448-L456）

```python
    async def _establish_hardware_connection(self) -> bool:
        """Establish connection to hardware (to be implemented by subclasses)."""
        # Placeholder implementation for testing
        return True

    async def _close_hardware_connection(self) -> None:
        """Close hardware connection (to be implemented by subclasses)."""
        # Placeholder implementation for testing
        pass
```

**证据要点**：连接建立与关闭均为 placeholder。

---

## E-03 原始数据读取仍含 placeholder 路径（代码证据）

**来源**：`v1/src/hardware/csi_extractor.py`（约 L458-L470）

```python
    async def _read_raw_data(self) -> bytes:
        ...
        if self.config.get('parser_format') == 'binary':
            return await self._read_udp_data()
        # Placeholder implementation for legacy text-mode testing
        return b"CSI_DATA:1234567890,3,56,2400,20,15.5,[1.0,2.0,3.0],[0.5,1.5,2.5]"
```

**证据要点**：非 binary 路径返回占位文本数据。

---

## E-04 Router 接口存在 mock_mode 与 Unknown 信息返回（代码证据）

**来源 1**：`v1/src/core/router_interface.py`（约 L142-L170）

```python
    def _generate_mock_csi_data(self) -> np.ndarray:
        ...
        return self._mock_csi_generator.generate()

    async def _collect_real_csi_data(self) -> Optional[np.ndarray]:
        ...
        raise RuntimeError(
            f"Real CSI data collection from router '{self.router_id}' requires "
            "hardware setup that is not configured..."
        )
```

**来源 2**：`v1/src/core/router_interface.py`（约 L221-L238）

```python
        if self.mock_mode:
            ...
            return self._mock_csi_generator.get_router_info()

        # For real routers, this would query the actual hardware
        return {
            "model": "Unknown",
            "firmware": "Unknown",
            "wifi_standard": "Unknown",
            ...
        }
```

**证据要点**：mock 与真实路径并存，真实信息探测默认 `Unknown`。

---

## E-05 姿态结果可由 mock 路径直接生成（代码证据）

**来源 1**：`v1/src/services/pose_service.py`（约 L236-L237）

```python
        if self.settings.mock_pose_data:
            return self._generate_mock_poses()
```

**来源 2**：`v1/src/services/pose_service.py`（约 L387-L404）

```python
    def _generate_mock_poses(self) -> List[Dict[str, Any]]:
        ...
        from src.testing.mock_pose_generator import generate_mock_poses
        return generate_mock_poses(max_persons=self.settings.pose_max_persons)
```

**来源 3**：`v1/src/services/pose_service.py`（约 L527-L540）

```python
                # Mock mode: generate mock poses directly (no fake CSI data)
                from src.testing.mock_pose_generator import generate_mock_poses
                ...
                result = {
                    "poses": mock_poses,
                    ...
                }
```

**证据要点**：在 mock 配置下可直接产生姿态结果。

---

## E-06 无 CSI 且非 mock 时抛未实现异常（代码证据）

**来源**：`v1/src/services/pose_service.py`（约 L508-L514）

```python
            if csi_data is None and not self.settings.mock_pose_data:
                raise NotImplementedError(
                    "Pose estimation requires real CSI data input. No CSI data was provided "
                    "and mock_pose_data is disabled..."
                )
```

**证据要点**：默认不具备“无前置即可推理”的完整链路。

---

## E-07 可运行性与验证链路未闭环（评审证据）

**来源 1**：`v1/docs/review/repo-runnability-review.md`（L7-L13）

```text
同步后仓库仍然是“有大量实现，但默认运行链路不稳定”的状态
- Python v1 主线：默认启动链路...阻断
- 可信验证链路：./verify ... FAIL
```

**来源 2**：`v1/docs/review/repo-runnability-review.md`（L72-L82）

```text
### 4) `verify` 仍未在当前环境复现 PASS
- `./verify` ... `VERDICT: FAIL`
- ...可验证真实性的核心叙事在普通环境下仍不稳
```

**证据要点**：默认运行与验证可复现性仍存在阻断。

---

## E-08 硬件专项评审指出 mock 与核心缺口（评审证据）

**来源 1**：`v1/docs/review/hardware-integration-review.md`（L30-L39）

```text
1. Mock implementation in production code
2. Missing implementation ... No actual hardware communication code
```

**来源 2**：`v1/docs/review/hardware-integration-review.md`（L60-L65）

```text
1. Mock implementation in production:
- `_parse_csi_response()` returns mock data
```

**来源 3**：`v1/docs/review/hardware-integration-review.md`（L233-L235）

```text
1. Mock implementations in production code - should be removed
2. Missing actual hardware communication - core functionality not implemented
```

**证据要点**：硬件链路仍未达到“真实生产链路”状态。

---

## E-09 跨环境泛化仍是提案状态（文档证据）

**来源**：`docs/adr/ADR-027-cross-environment-domain-generalization.md`（L1-L8）

```text
# ADR-027: Project MERIDIAN ...
| **Status** | Proposed |
```

**证据要点**：关键高难能力属于规划/提案，不是已落地能力。

---

## E-10 内部叙事存在“production-ready”与“仍需补核心”并存（评审证据）

**来源 1**：`v1/docs/review/comprehensive-system-review.md`（L7-L10）

```text
Overall Assessment: PRODUCTION-READY
... ready for deployment with minor configuration adjustments
```

**来源 2**：`v1/docs/review/comprehensive-system-review.md`（L88-L92）

```text
2. Mock Data Removal: Remove mock implementations from production code
4. Hardware Implementation: Complete actual hardware communication code
```

**证据要点**：平台级 readiness 与核心能力 readiness 被混合表述。

---

## 附：使用说明

- 本附件用于支撑正文中的 E-01 ~ E-10 结论。
- 引用建议：正文写观点，附件放原文摘录，避免正文过长影响可读性。
- 如需对外发布版本，建议把代码摘录改为“截图 + hash + commit id”形式，增强审计可追溯性。
